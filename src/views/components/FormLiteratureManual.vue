<template>
  <div>
    <div>
      <md-field class="md-size-100">
        <md-select v-model="litAuswahl">
          <md-option value="Buch">Buch</md-option>
          <md-option value="Artikel">Artikel</md-option>
          <md-option value="DigitalesDokument">DigitalesDokument</md-option>
        </md-select>
      </md-field>
    </div>
    <!-- END DropDown -->
    <!-- Ausgabe der Daten in Formular -->
    <div @change="generateQuery">
      <!-- Titel -->
      <div class="md-size-100">
        <md-field>
          <label>Titel*</label>
          <md-input v-model="inputs.titelNeu" />
        </md-field>
      </div>
      <!-- ISBN -->
      <div v-if="litAuswahl == 'Buch'" class="md-size-100">
        <md-field>
          <label>ISBN</label>
          <md-input v-model="inputs.isbnNeu" @change="getOpacLink" />
        </md-field>
      </div>
      <!-- Erscheinen IN z.B. Journal-->
      <div v-if="litAuswahl == 'Artikel'" class="md-size-100">
        <label>In</label>
        <div class="md-layout md-gutter">
          <!-- Titel_IN -->
          <div class="md-layout-item md-size-60">
            <md-field>
              <label>Titel</label>
              <md-input v-model="inputs.titelInBandNeu" />
            </md-field>
          </div>
          <!-- Band_IN -->
          <div class="md-layout-item md-size-20">
            <md-field>
              <label>Band</label>
              <md-input v-model="inputs.bandInBandNeu" />
            </md-field>
          </div>
          <!-- Jahr_IN -->
          <div class="md-layout-item md-size-20">
            <md-field>
              <label>Jahr</label>
              <md-input v-model="inputs.jahrInBandNeu" />
            </md-field>
          </div>
        </div>
      </div>
      <!-- Herausgeber/ Verlag -->
      <div class="md-size-100">
        <md-field>
          <label>Herausgeber/ Verlag</label>
          <md-input v-model="inputs.publisherNeu" />
        </md-field>
      </div>

      <!-- Veröffentlichung -->
      <div class="md-size-100">
        <md-field>
          <label>Veröffentlichung*</label>
          <md-input v-model="inputs.datePublishedNeu" />
        </md-field>
      </div>
      <!-- Auflage -->
      <div v-if="litAuswahl == 'Buch'" class="md-size-100">
        <md-field>
          <label>Auflage</label>
          <md-input v-model="inputs.auflageNeu" />
        </md-field>
      </div>
      <!-- Artikel Seiten "von" - "bis" z.B. Journal-->
      <div v-if="litAuswahl == 'Artikel'" class="md-size-100">
        <div class="md-layout md-gutter">
          <!-- Seiten von -->
          <div class="md-layout-item md-size-50">
            <md-field>
              <label>Seiten von</label>
              <md-input v-model="inputs.seitenVonInBandNeu" />
            </md-field>
          </div>
          <!-- Seiten bis -->
          <div class="md-layout-item md-size-50">
            <md-field>
              <label>Seiten bis</label>
              <md-input v-model="inputs.seitenBisInBandNeu" />
            </md-field>
          </div>
        </div>
      </div>
      <!-- URL z.B. OPAC-Link, Springer-Link oder arxiv-Link -->
      <div class="md-size-100">
        <md-field>
          <label v-if="loading">Suche OPAC-Link ...</label>
          <label v-else>URL</label>
          <md-input v-model="inputs.urlLinkNeu" v-if="loading" disabled />
          <md-input v-model="inputs.urlLinkNeu" v-else />
        </md-field>
      </div>

      <!-- DOI-Link (Identifier nicht ausgeben, aber DOI-Link, wenn vorhanden) -->
      <div v-if="litAuswahl == 'Buch'" class="md-size-100">
        <md-field>
          <label>DOI</label>
          <md-input v-model="inputs.doiLinkNeu" />
        </md-field>
      </div>
      <!-- Autoren/innen -->
      <div class="md-size-100">
        <label>Autoren/innen</label>
        <div
          v-for="(autor, i) in autoren"
          :key="autor"
          class="md-layout md-gutter"
        >
          <!-- Nachname -->
          <div class="md-layout-item md-size-20">
            <md-field>
              <label>Nachname*</label>
              <md-input v-model="autor.autorNachnameNeu" />
            </md-field>
          </div>
          <!-- Vorname -->
          <div class="md-layout-item md-size-20">
            <md-field>
              <label>Vorname*</label>
              <md-input v-model="autor.autorVornameNeu" />
            </md-field>
          </div>
          <!-- URL/ Profil-Link -->
          <div class="md-layout-item md-size-60">
            <md-field>
              <label>Profil-Link/URL*</label>
              <md-input
                v-model="autor.autorProfilLinkNeu"
                @change="checkAutor(i)"
              />
              <AutorSelectionPopUp
                v-if="showPopUp"
                @close="showPopUp = false"
                @duplicateChecked="handleAutorSelection"
                :existingAuthors="existingAuthors"
                :autorIndex="authorIndexPopUp"
              >
              </AutorSelectionPopUp>
              <!-- Plus, Minus, Verschieben-Symbole -->
              <span>
                <i
                  class="fas fa-minus-circle"
                  @click="removeAutor('autoren', i)"
                  v-show="autoren.length > 1"
                />
                <i
                  class="fas fa-plus-circle"
                  @click="addAutor('autoren', i)"
                  v-show="i == autoren.length - 1"
                />
                <i class="handle fas fa-arrows-alt" style="margin-left: 10px" />
              </span>
            </md-field>
          </div>
        </div>
      </div>
      <!-- ENDE Autoren/innen -->
    </div>
    <!-- ENDE - Ausgabe der Daten in Formular -->
  </div>
</template>

<script>
import { v4 as uuidv4 } from "uuid";
import axios from "axios";
import AutorSelectionPopUp from "./AutorSelectionPopUp";

export default {
  props: ["moduleUri"],
  name: "literatureManual",
  components: {
    AutorSelectionPopUp,
  },
  data() {
    return {
      changedArray: [],
      label: "Manuell",
      litAuswahl: "Buch",
      prefixes:
        "PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> " +
        "PREFIX module: <https://bmake.th-brandenburg.de/module/> " +
        "PREFIX schema: <https://schema.org/>  ",
      inputs: {
        id: 0,
        titelNeu: [],
        isbnNeu: [],
        publisherNeu: [],
        datePublishedNeu: [],
        auflageNeu: [],
        urlLinkNeu: [],
        doiLinkNeu: [],
        titelInBandNeu: [],
        bandInBandNeu: [],
        jahrInBandNeu: [],
        seitenVonInBandNeu: [],
        seitenBisInBandNeu: [],
        literaturUri: [],
        literaturJournalUri: [],
        publisherUri: [],
      },
      authorIndexPopUp: 0,
      autoren: [
        {
          autorUri: [],
          autorNachnameNeu: [],
          autorVornameNeu: [],
          autorProfilLinkNeu: [],
        },
      ],
      loading: false,
      existingAuthors: [],
      showPopUp: false,
    };
  },
  methods: {
    // Add author entry
    addAutor(input, index) {
      this[input].push({ autorNachnameNeu: [] });
    },
    // Remove author entry
    removeAutor(input, index) {
      this[input].splice(index, 1);
    },
    // Check if author with same name exists
    checkAutor(index) {
      let autor = this.autoren[index];

      // User has to input all values for a check
      if (
        autor.autorVornameNeu.length < 1 ||
        autor.autorNachnameNeu.length < 1 ||
        autor.autorProfilLinkNeu.length < 1
      )
        return;

      let queryAutor = this.prefixes;

      queryAutor +=
        " SELECT DISTINCT ?autorUri ?autorVorname ?autorNachname ?autorProfilLink ";
      queryAutor += " WHERE { ";
      queryAutor += " ?autorUri  a module:Author; ";
      queryAutor += "            schema:givenName ?autorVorname; ";
      queryAutor += "            schema:familyName ?autorNachname. ";

      queryAutor += " OPTIONAL { ";
      queryAutor += " ?autorUri schema:sameAs ?autorProfilLink.  ";
      queryAutor += " } ";

      queryAutor +=
        " {?autorUri schema:sameAs '" + autor.autorProfilLinkNeu + "' } ";
      queryAutor += " UNION ";
      queryAutor +=
        " { ?autorUri schema:familyName '" + autor.autorNachnameNeu + "' ; ";
      queryAutor +=
        "             schema:givenName '" + autor.autorVornameNeu + "' . } ";
      queryAutor += "  }";

      // Daten vom Fuseki abrufen
      axios
        .post(
          "http://fbwsvcdev.fh-brandenburg.de:8080/fuseki/modcat/query",
          queryAutor,
          {
            headers: { "Content-Type": "application/sparql-query" },
          }
        )
        .then((response) => {
          // JSON responses are automatically parsed.
          let result = response.data.results.bindings;
          let formattedResult = [];
          // Reformat author results
          for (let entry of result) {
            formattedResult.push({
              given: entry.autorVorname.value,
              family: entry.autorNachname.value,
              url: entry.autorProfilLink.value,
              uri: entry.autorUri.value,
            });
          }
          this.existingAuthors = formattedResult;
          this.authorIndexPopUp = index;
        })
        .catch((e) => {
          this.errors.push(e);
        });
    },
    // Get OPAC link for isbn
    getOpacLink() {
      this.inputs.urlLinkNeu = "";
      this.loading = true;
      axios
        .get(
          "https://opac.th-brandenburg.de/search?isbn=" + this.inputs.isbnNeu,
          {
            headers: {
              Accept: "text/html",
            },
            crossdomain: true,
          }
        )
        .then((response) => {
          if (response.data.includes("in die Merkliste")) {
            this.inputs.urlLinkNeu =
              "https://opac.th-brandenburg.de/search?isbn=" +
              this.inputs.isbnNeu;
          }
          this.loading = false;
        })
        .catch((e) => {
          this.loading = false;
        });
    },
    // Handle selection from duplicate popup
    handleAutorSelection(result) {
      if (result.autor.given.length > 0) {
        this.autoren[result.index].autorVornameNeu = result.autor.given;
        this.autoren[result.index].autorNachnameNeu = result.autor.family;
        this.autoren[result.index].autorProfilLinkNeu = result.autor.url;
        this.autoren[result.index].autorUri = result.autor.uri;
      }
    },
    // Query for inserting new literature
    generateQuery() {
      let query = ""; //let query = this.prefixes; // wird in LiteratureForm hinzugefügt

      // Generate Literature URI
      if (this.inputs.doiLinkNeu.length > 0) {
        this.inputs.literaturUri = "<" + this.inputs.doiLinkNeu + ">";
      } else if (this.inputs.isbnNeu.length > 0) {
        this.inputs.literaturUri =
          "<http://isbn-international.org/" + this.inputs.isbnNeu + ">";
      } else {
        this.inputs.literaturUri =
          "<https://th-brandenburg.de/literatur/" + uuidv4() + ">";
      }

      // Generate Autoren URIs
      for (let autor of this.autoren) {
        // if (autor.autorProfilLinkNeu.inclued('orcid.org')) -> Dann Orcid als URI
        autor.autorUri = "<https://th-brandenburg.de/autor/" + uuidv4() + ">";
      }

      // Generate Pulisher URIs (Herausgeber/ Verlag)
      if (this.inputs.publisherNeu.length > 0) {
        this.inputs.publisherUri =
          "<https://th-brandenburg.de/publisher/" + uuidv4() + ">";
      }

      if (this.inputs.titelNeu.length > 0) {
        //query += " INSERT { "; //wird in LiteratureForm hinzugefügt
        //query += "module:GPMO "; //Nur zum Test
        query += "  <" + this.moduleUri + "> ";
        query += "schema:citation " + this.inputs.literaturUri + " . ";

        if (this.litAuswahl === "Buch") {
          query += this.inputs.literaturUri + " a schema:Book ; ";

          if (this.inputs.isbnNeu.length > 0) {
            query += 'schema:isbn "' + this.inputs.isbnNeu + '"; ';
          }
          if (this.inputs.publisherNeu.length > 0) {
            // Referenz zur Pulisher URIs erzeugen (Herausgeber/ Verlag)
            query += "schema:publisher " + this.inputs.publisherUri + "; ";
          }
          if (this.inputs.datePublishedNeu.length > 0) {
            query +=
              'schema:datePublished "' + this.inputs.datePublishedNeu + '"; ';
          }
          if (this.inputs.auflageNeu.length > 0) {
            query += 'schema:bookEdition "' + this.inputs.auflageNeu + '"; ';
          }
          if (this.inputs.urlLinkNeu.length > 0) {
            query += 'schema:url "' + this.inputs.urlLinkNeu + '"; ';
          }
          if (this.inputs.doiLinkNeu.length > 0) {
            query += 'schema:identifier "' + this.inputs.doiLinkNeu + '"; ';
          }
          //Referenz zu den Autoren in Lit erzeugen
          if (
            this.autoren.every((autor) => autor.autorNachnameNeu.length > 0)
          ) {
            query += "schema:author ";
            for (let autor of this.autoren) {
              query += autor.autorUri + " , ";
            }
            query = query.slice(0, query.length - 3);
            query += " ; ";
          }
        } else if (this.litAuswahl === "Artikel") {
          //In Journal als Book definiert
          if (this.inputs.titelInBandNeu.length > 0) {
            this.inputs.literaturJournalUri =
              "<https://th-brandenburg.de/literatur/" + uuidv4() + ">";
            query += this.inputs.literaturJournalUri + " a schema:Book ; ";

            if (this.inputs.bandInBandNeu.length > 0) {
              query +=
                'schema:bookEdition "' + this.inputs.bandInBandNeu + '"; ';
            }
            if (this.inputs.jahrInBandNeu.length > 0) {
              query +=
                'schema:datePublished "' + this.inputs.jahrInBandNeu + '"; ';
            }

            query += 'schema:headline "' + this.inputs.titelInBandNeu + '". ';
          }

          //Artikle
          query += this.inputs.literaturUri + " a schema:Article ; ";

          if (this.inputs.titelInBandNeu.length > 0) {
            query +=
              'schema:isPartOf "' + this.inputs.literaturJournalUri + '"; ';
          }
          if (this.inputs.publisherNeu.length > 0) {
            // Referenz zur Pulisher URIs erzeugen (Herausgeber/ Verlag)
            query += "schema:publisher " + this.inputs.publisherUri + "; ";
          }
          if (this.inputs.datePublishedNeu.length > 0) {
            query +=
              'schema:datePublished "' + this.inputs.datePublishedNeu + '"; ';
          }
          if (
            this.inputs.seitenVonInBandNeu.length > 0 &&
            this.inputs.seitenBisInBandNeu.length > 0
          ) {
            query +=
              'schema:pageStart "' + this.inputs.seitenVonInBandNeu + '"; ';
            query +=
              'schema:pageEnd "' + this.inputs.seitenBisInBandNeu + '"; ';
          }
          if (this.inputs.urlLinkNeu.length > 0) {
            query += 'schema:url "' + this.inputs.urlLinkNeu + '"; ';
          }
          //Referenz zu den Autoren in Lit erzeugen
          if (
            this.autoren.every((autor) => autor.autorNachnameNeu.length > 0)
          ) {
            query += "schema:author ";
            for (let autor of this.autoren) {
              query += autor.autorUri + " , ";
            }
            query = query.slice(0, query.length - 3);
            query += " ; ";
          }
        } else if (this.litAuswahl === "DigitalesDokument") {
          query += this.inputs.literaturUri + " a schema:DigitalDocument ; ";

          if (this.inputs.publisherNeu.length > 0) {
            query += "schema:publisher " + this.inputs.publisherUri + "; ";
          }
          if (this.inputs.datePublishedNeu.length > 0) {
            query +=
              'schema:datePublished "' + this.inputs.datePublishedNeu + '"; ';
          }
          if (this.inputs.urlLinkNeu.length > 0) {
            query += 'schema:url "' + this.inputs.urlLinkNeu + '"; ';
          }
          //Referenz zu den Autoren in Lit erzeugen
          if (
            this.autoren.every((autor) => autor.autorNachnameNeu.length > 0)
          ) {
            query += "schema:author ";
            for (let autor of this.autoren) {
              query += autor.autorUri + " , ";
            }
            query = query.slice(0, query.length - 3);
            query += " ; ";
          }
        }
        //Titel
        query += 'schema:headline "' + this.inputs.titelNeu + '". ';

        //Autoren/innen
        if (this.autoren.every((autor) => autor.autorNachnameNeu.length > 0)) {
          for (let autor of this.autoren) {
            query += autor.autorUri + " a module:Author ; ";

            if (autor.autorNachnameNeu != "") {
              query += 'schema:familyName "' + autor.autorNachnameNeu + '"; ';
            }
            if (autor.autorVornameNeu != "") {
              query += 'schema:givenName "' + autor.autorVornameNeu + '"; ';
            }

            query += 'schema:sameAs "' + autor.autorProfilLinkNeu + '". ';
          }
        }

        //Herausgeber/ Verlag
        if (this.inputs.publisherNeu.length > 0) {
          // Generate Pulisher URIs (Herausgeber/ Verlag)
          query += this.inputs.publisherUri + " a schema:Organization ; ";
          query += 'schema:legalName "' + this.inputs.publisherNeu + '". ';
        }
      }

      this.updateQuery = query;

      // Let Literature.vue know of changes
      this.$emit("queryChanged", query);
    },
  },
  watch: {
    existingAuthors() {
      if (this.existingAuthors.length < 1) return;
      this.showPopUp = true;
    },
  },
};
</script>
