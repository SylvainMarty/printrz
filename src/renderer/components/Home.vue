<template>
  <div>
    <md-empty-state
      v-if="printers.length === 0"
      md-rounded
      md-icon="print"
      md-label="No printer configured"
      md-description="You must configure at least one printer on your computer.">
      <md-button class="md-primary md-raised" @click="fetchPrinters">Refresh</md-button>
    </md-empty-state>

    <template v-else>
      <md-subheader>
        <md-button @click="fetchPrinters" class="md-icon-button">
          <md-icon>refresh</md-icon>
        </md-button>
      </md-subheader>
      <md-card v-for="pr in printers" :key="pr.name" md-with-hover>
        <md-card-header>
          <md-card-media>
            <md-icon class="md-size-3x">print</md-icon>
          </md-card-media>
        
          <md-card-header-text>
            <div class="md-title">
              <template v-if="pr.options">{{ pr.options['printer-info'] }}</template>
              <template v-else>{{ pr.name }}</template>
            </div>
            <div class="md-subhead">{{ pr.name }}</div>
          </md-card-header-text>
        </md-card-header>

        <md-card-actions>
          <md-button @click="showDetails(pr)">Details</md-button>
          <md-button class="md-accent" @click="printTest(pr)">Print test</md-button>
        </md-card-actions>
      </md-card>

      <md-dialog :md-active.sync="showDialog">
        <md-dialog-title>Preferences</md-dialog-title>

        <md-dialog-content>
          <pre>{{ printerDetails }}</pre>
        </md-dialog-content>

        <md-dialog-actions>
          <md-button class="md-primary" @click="showDialog = false">Close</md-button>
        </md-dialog-actions>
      </md-dialog>
    </template>
  </div>
</template>

<script>
  import Vue from 'vue'
  import { MdCard, MdButton, MdDialog, MdEmptyState, MdSubheader } from 'vue-material/dist/components'

  Vue.use(MdCard)
  Vue.use(MdButton)
  Vue.use(MdDialog)
  Vue.use(MdEmptyState)
  Vue.use(MdSubheader)

  export default {
    name: 'home',
    data: function () {
      return {
        printers: [],
        showDialog: false,
        printerDetails: {}
      }
    },

    created: function () {
      this.fetchPrinters()
    },

    methods: {
      fetchPrinters () {
        this.$http.get('/printers').then(response => {
          this.printers = response.data
        })
      },
      showDetails (printer) {
        this.printerDetails = printer
        this.showDialog = true
      },
      printTest (printer) {
        // Set content to print...
        // Create ESP/POS commands for sample label
        var esc = '\x1B' // ESC byte in hex notation
        var newLine = '\x0A' // LF byte in hex notation
        var cmds = esc + '@' + newLine // Initializes the printer (ESC @)
        cmds += esc + '!' + '\x38' // Emphasized + Double-height + Double-width mode selected (ESC ! (8 + 16 + 32)) 56 dec => 38 hex
        cmds += 'BEST DEAL STORES' // text to print
        cmds += newLine + newLine
        cmds += esc + '!' + '\x00' // Character font A selected (ESC ! 0)
        cmds += 'COOKIES                   5.00'
        cmds += newLine
        cmds += 'MILK 65 Fl oz             3.78'
        cmds += newLine + newLine
        cmds += 'SUBTOTAL                  8.78'
        cmds += newLine
        cmds += 'TAX 5%                    0.44'
        cmds += newLine
        cmds += 'TOTAL                     9.22'
        cmds += newLine
        cmds += 'CASH TEND                10.00'
        cmds += newLine
        cmds += 'CASH DUE                  0.78'
        cmds += newLine + newLine
        cmds += esc + '!' + '\x18' // Emphasized + Double-height mode selected (ESC ! (16 + 8)) 24 dec => 18 hex
        cmds += '# ITEMS SOLD 2'
        cmds += esc + '!' + '\x00' // Character font A selected (ESC ! 0)
        cmds += newLine + newLine
        cmds += '11/03/13  19:53:17'
        cmds += newLine + newLine + newLine + newLine
        this.$http.post('/job', {
          printer: printer.name,
          type: 'RAW',
          data: cmds
        })
      }
      /* reload () {
        this.$electron.remote.getGlobal('printrz').api.stopServer().finally(() => {
          this.$electron.remote.getGlobal('printrz').api = new Api(() => {
            this.fetchPrinters()
          })
        })
      } */
    }
  }
</script>


<style lang="scss" scoped>
  .md-card {
    margin-bottom: 12px;
  }
</style>