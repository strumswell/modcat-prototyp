<template>
  <div>
    <!-- Tabs -->
    <label>Hier ist eine Menge an Literatur</label>

    <!-- Tabs -->
    <div>
      <button
        v-for="tab in tabs"
        :key="tab"
        :class="['tab-button', { active: currentTab === tab }]"
        @click="currentTab = tab"
      >
        {{ tab }}
      </button>
      <component :is="currentTabComponent" class="tab" />
    </div>
    <div>
        <!-- Hier soll die SPARQL-Ausgabe hin -->
        <div class="md-layout-item md-size-80">
            <md-field>
              <label>Modulbezeichnung</label>
              <md-input
                v-if="modLiterature.length > 0 && existence"
                v-model="modLiterature[0].label.value"
                disabled
              />
              <md-input v-else disabled />
            </md-field>
        </div>
        <div> <!-- Simple Tabelle => Nur Test, am Ende wieder löschen! -->
          <table style="width:100%">
            <tr>
              <td><md-field>
                <label>Titel</label>
                <md-input v-model="modLiterature[0].titel.value" /></md-field></td>
              <td><md-field>
                <label>Auflage</label>
                <md-input v-model="modLiterature[0].auflage.value" /></md-field></td>
              <td><md-field>
                <label>Autor</label>
                <md-input v-model="modLiterature[0].autorLabel.value" /></md-field></td>
            </tr>
            <tr>
              <td><md-field><md-input v-model="modLiterature[1].titel.value" /></md-field></td>
              <td><md-field><md-input v-model="modLiterature[1].auflage.value" /></md-field></td>
              <td><md-field><md-input v-model="modLiterature[1].autorLabel.value" /></md-field></td>
            </tr>
            <tr>
              <td><md-field><md-input v-model="modLiterature[2].titel.value" /></md-field></td>
              <td><md-field><md-input v-model="modLiterature[2].auflage.value" /></md-field></td>
              <td><md-field><md-input v-model="modLiterature[2].autorLabel.value" /></md-field></td>
            </tr>
          </table>
        </div>

        <div> <!-- Text-Ausgabe JSON-->
          <span v-once>JSON-Ausgabe: 
            {{ this.modLiterature }}</span>
        </div>
              
    
    </div>
  </div>
</template>

<script>
import FormLiteratureDOI from "./FormLiteratureDOI";
import FormLiteratureISBN from "./FormLiteratureISBN";
import FormLiteratureManual from "./FormLiteratureManual";

console.log(FormLiteratureDOI);

export default {
  components: {
    FormLiteratureDOI,
    FormLiteratureISBN,
    FormLiteratureManual,
  },
  props: [
    "modLiteratureOrigin",
    "moduleUri",
    "role",
    "newBoolean",
    "code",
    "basicFilled ",
  ],
  name: "literature",
  mixins: [],
  directives: {},
  data() {
    return {
      modLiterature: [],
      currentTab: "DOI",
      tabs: ["DOI", "ISBN", "Manual"],
    };
  },
  computed: {
    currentTabComponent() {
      return "FormLiterature" + this.currentTab;
    },
  },
  mounted() {
    this.initialState();
    // Log-Ausgabe
    console.log(this.modLiterature);
  },
  methods: { 
    initialState() {
      
      this.existence = true;
      this.modLiterature = [];
      this.delete = [];
      this.insert = [];
      this.where = [];
      this.modLiterature = _.cloneDeep(this.modLiteratureOrigin);
      //if (this.newBoolean) {
      // this.checkModule();
      //}
      //this.$v.$reset();
    },

    updateData() {
    // für SPARQL-DataUpdate 
    
    /*axios
        .post(
          "http://fbwsvcdev.fh-brandenburg.de:8080/fuseki/modcat/update",
          query,
          {
            headers: { "Content-Type": "application/sparql-update" }
          }
        )
        .then(response => {
          let status = response.status;
          if (status == 204) {
            this.notification = true;
            setTimeout(() => {
              this.notification = false;
            }, 2000);
          }
          this.clearCache();
        })
        .catch(e => {
          this.errors.push(e);
        }); */
    },
      
  },
  watch: {
    modLiteratureOrigin(v) {
      this.initialState();
    },
        
  }
};
</script>


<style scoped>
.tab-button {
  padding: 6px 10px;
  border-top-left-radius: 3px;
  border-top-right-radius: 3px;
  border: 1px solid #ccc;
  cursor: pointer;
  background: #f0f0f0;
  margin-bottom: -1px;
  margin-right: -1px;
}
.tab-button:hover {
  background: #e0e0e0;
}
.tab-button.active {
  background: #e0e0e0;
}
.tab {
  border: 1px solid #ccc;
  padding: 10px;
}
</style>
