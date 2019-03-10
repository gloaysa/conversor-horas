<template>
  <div class="section">
    <div class="column">
      <h1 class="title is-1">Conversor archivo CSV</h1>
    </div>

    <div class="columns container is-centered">

      <div class="column is-half">
        <div class="field">
          <label class="label">Name of employee</label>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <input class="input" type="text" v-model="name" placeholder="Name Surname">
              </div>
            </div>
          </div>
        </div>
        <div class="field">
          <label class="label">Module (Product)</label>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <input class="input" type="text" v-model="product" placeholder="Module (e.g. Connect MB)">
              </div>
            </div>
          </div>
        </div>
        <div class="field">
          <label class="label">Role</label>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <input class="input" type="text" v-model="role" placeholder="Role (e.g. DEV-FE)">
              </div>
            </div>
          </div>
        </div>
        <div class="field">
          <label class="label">Supplier</label>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <input class="input" type="text" v-model="supplier" placeholder="Supplier (e.g. Novatec Consulting Gmbh)">
              </div>
            </div>
          </div>
        </div>

        <div class="file has-name is-boxed is-centered"
             v-bind:class="{'is-info': !loadedFile, 'is-primary': loadedFile}"
             v-show="name.length > 1 && supplier.length > 1">
          <label class="file-label">
            <input class="file-input" type="file"
                   id="fileInput"
                   @change="upload">
            <span class="file-cta">
              <span class="file-icon">
                <i class="fas fa-upload"></i>
              </span>
              <span class="file-label">
                Selecciona archivo...
              </span>
            </span>
            <span class="file-name" id="file-name" v-show="loadedFile"></span>
          </label>
        </div>

      </div>

      <div class="column is-half">
        <div class="control">
        <textarea
          class="textarea"
          v-bind:class="{'is-warning': isEditable}"
          rows="17"
          readonly
          v-model='doc'>
        </textarea>
        </div>
        <div class="buttons are-large is-fullwidth">
            <button
              class="button is-primary"
              @click='save'
              v-if="loadedFile">Guardar
            </button>

        </div>


      </div>
    </div>
  </div>
</template>

<script>
  import Papa from 'papaparse'
  import Blob from 'blob'
  import FileSaver from 'file-saver'

  export default {
    name: 'conversor',
    data () {
      return {
        doc: null,
        loadedFile: false,
        isEditable: false,
        name: '',
        product: '',
        role: '',
        supplier: ''
      }
    },
    methods: {
      upload (e) {
        const that = this
        const fileToLoad = event.target.files[0]
        const reader = new FileReader()
        document.getElementById('file-name').innerHTML = event.target.files[0].name ? event.target.files[0].name : 'Selecciona archivo'
        this.loadedFile = true
        reader.onload = fileLoadedEvent => {
          Papa.parse(fileLoadedEvent.target.result, {
            header: false,
            skipEmptyLines: true,
            complete (results) {
              const HEADER = ['Hours On-Site', 'Hours Off-Site', 'Name of employee', 'Date', 'PT', 'Module (Product)', 'Role', 'Comment', 'Dayrate', 'Supplier', 'Euro']
              let data = results.data
              results.data.forEach((dataHour, index) => {
                let dataIndex = data[index]
                let date = dataHour[0]
                let hour = dataHour[5].replace(',', '.')
                if (getOnSiteHour(dataHour[4])) {
                  dataIndex[0] = hour
                  dataIndex[1] = '0'
                } else {
                  dataIndex[1] = hour
                  dataIndex[0] = '0'
                }
                dataIndex[2] = that.name
                dataIndex[3] = date
                dataIndex[4] = ''
                dataIndex[5] = that.product
                dataIndex[6] = that.role
                dataIndex[7] = ''
                dataIndex[8] = ''
                dataIndex[9] = that.supplier
                dataIndex[10] = 0
              })
              data.unshift(HEADER)
              that.doc = Papa.unparse(data)

              function getOnSiteHour (hours) {
                return !!hours.match(/remote/)
              }
            },
            error (errors) {
              console.log('error', errors)
            }
          })
        }
        reader.readAsText(fileToLoad)
      },
      save () {
        let that = this
        const blob = new Blob([this.parseJSONtoCSV()], { type: 'text/csv' })
        let title = createTitle()
        FileSaver.saveAs(blob, title + '.csv')
        function createTitle () {
          let supplier = that.supplier.replace(' ', '_')
          let name = that.name.split(' ')
          let date = new Date()
          let month = date.getMonth() + 1
          let newM = month < 10 ? ('0' + month) : month
          let year = date.getUTCFullYear()
          return year + newM + '_external_report_' + supplier + '_' + name[1] + '_' + name[0]
        }
      },
      parseJSONtoCSV () {
        return this.doc
      }
    }
  }
</script>


