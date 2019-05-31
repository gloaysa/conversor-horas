<template>
  <div class="section">
    <div class="column">
      <h1 class="title is-1">Conversor archivo CSV</h1>
        <p>Los datos aquí introducidos aparecerán en sus correspondientes columnas.</p>
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

        <div class="field">
          <input type="checkbox" id="checkbox" v-model="onRemoteHours">
          <label for="checkbox"> Different hourly rate on remote? </label>
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
        name: 'Guillermo Loaysa',
        product: 'Connect MB',
        role: 'DEV-FE',
        supplier: 'Novatec Consulting',
        onRemoteHours: true
      }
    },
    methods: {
      upload (e) {
        const that = this;
        const fileToLoad = event.target.files[0];
        const reader = new FileReader();

        document.getElementById('file-name').innerHTML = event.target.files[0].name ? event.target.files[0].name : 'Selecciona archivo';
        this.loadedFile = true;

        reader.onload = fileLoadedEvent => {
          Papa.parse(fileLoadedEvent.target.result, {
            header: false,
            skipEmptyLines: true,
            
            complete (results) {
              const HEADERS = [
                'Hours On-Site',
                'Hours Off-Site',
                'Name of employee',
                'Date',
                'PT',
                'Module (Product)',
                'Role',
                'Comment',
                'Dayrate',
                'Supplier',
                'Euro'
              ];
              let table = results.data;

              table.forEach(row => {
                const date = changeDate(row[0]);
                that.onRemoteHours ? splitOnSiteAndOffSiteHours(row) : noSplitHours(row);
                
                row[2] = that.name; // Name of employee
                row[3] = date; // Date
                row[4] = ''; // PT
                row[5] = that.product; // Product
                row[6] = that.role; // Role
                row[7] = ''; // Comment
                row[8] = ''; // DayRate
                row[9] = that.supplier; // Supplier
                row[10] = 0; // Euro

                that.onRemoteHours ? '' : row.shift();
              });
              that.onRemoteHours ? '' : HEADERS.shift();
              table.unshift(HEADERS);

              const papaParseSaveConfig = {
                delimiter: ';',
                skipEmptyLines: true
              };
              that.doc = Papa.unparse(table, papaParseSaveConfig);

              function noSplitHours (row) {
                HEADERS[1] = 'Hours';
                row[1] = row[5];
              }

              function splitOnSiteAndOffSiteHours (row) {
                let hour = row[5];

                if (isThisHourOnSite(row[4])) {
                  row[1] = hour; // Off-site
                  row[0] = ''; // On-site
                } else {
                  row[0] = hour; // On-site
                  row[1] = ''; // Off-site
                }
              }

              function changeDate (date) {
                const newDate = new Date(date).toLocaleDateString('es-ES').replace(/\//g, '.');
                return newDate;
              }

              function isThisHourOnSite (hours) {
                return !!hours.match(/remote/);
              }
            },
            error (errors) {
              console.error('error', errors);
            }
          })
        }
        reader.readAsText(fileToLoad);
      },
      save () {
        const that = this;
        const blob = new Blob([this.parseJSONtoCSV()], { type: 'text/csv' });
        const title = createTitle();

        FileSaver.saveAs(blob, title + '.csv');

        function createTitle () {
          const supplier = that.supplier.replace(' ', '_');
          const name = that.name.split(' ');
          const date = new Date();
          const month = date.getMonth() + 1;
          const newM = month < 10 ? ('0' + month) : month;
          const year = date.getUTCFullYear();
          return year + newM + '_external_report_' + supplier + '_' + name[1] + '_' + name[0];
        }
      },
      parseJSONtoCSV () {
        return this.doc;
      }
    }
  }
</script>


