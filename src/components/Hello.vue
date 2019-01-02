<template>
  <v-container fluid>
    <v-progress-linear color="black" v-if="loading == true" :indeterminate="true"></v-progress-linear>
    <v-btn
      color="white"
      light
      small
      absolute
      bottom
      left
      fab
      @click="logout"
      style="bottom: 60px; left:60px;"
    >
      <v-icon>arrow_back</v-icon>
    </v-btn>
    <v-btn
      color="black"
      dark
      small
      absolute
      bottom
      left
      fab
      @click="openSheetNew"
      style="bottom: 60px; left:94%;"
    >
      <v-icon>add</v-icon>
    </v-btn>
    <v-dialog dark v-if="dialogIssueSolusi" persistent v-model="dialogIssueSolusi">
      <v-card>
        <v-card-title class="title grey darken-2" primary-title>
          Tambahkan Issue dan Solusi Komputer
        </v-card-title>
        <v-card-text>
          <v-autocomplete
              v-model="selectedIssue"
              :items="issue"
              box
              chips
              color="grey lighten-2"
              label="Issue"
              item-text="issue"
              item-value="idIssue"
              @focus="fetchIssue"
            >
              <template
                slot="selection"
                slot-scope="data"
              >
                <v-chip
                  :selected="data.selected"
                  class="chip--select-multi"
                  @input="remove(data.item)"
                >
                  <v-avatar color="grey" style="color: white;">
                    {{ data.item.warnText }}
                  </v-avatar>
                  {{ data.item.issue }}                  
                </v-chip>
              </template>
              <template
                slot="item"
                slot-scope="data"
              >
                <template v-if="typeof data.item !== 'object'">
                  <v-list-tile-content v-text="data.item"></v-list-tile-content>
                </template>
                <template v-else>
                  <v-list-tile-content>
                    {{ data.item.issue }}
                    <v-chip>
                    Warn Level : {{ data.item.warnText }}
                  </v-chip>
                  </v-list-tile-content>
                </template>
              </template>
            </v-autocomplete>
            <v-autocomplete
              v-model="selectedSolusi"
              :items="solusi"
              box
              chips
              color="grey lighten-2"
              label="Solusi"
              item-text="solusi"
              item-value="idSolusi"
              @focus="fetchSolusi"
              multiple
            >
              <template
                slot="selection"
                slot-scope="data"
              >
                <v-chip>
                {{ data.item.solusi }}, RP. {{ data.item.harga }}
                </v-chip>
              </template>
              <template
                slot="item"
                slot-scope="data"
              >
                <template v-if="typeof data.item !== 'object'">
                  <v-list-tile-content v-text="data.item"></v-list-tile-content>
                </template>
                <template v-else>
                  <v-list-tile-content>
                    {{ data.item.solusi }}
                    <v-chip>
                    Harga : RP.{{ data.item.harga }}
                  </v-chip>
                  </v-list-tile-content>
                </template>
              </template>
            </v-autocomplete>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              light
              color="white"
              @click="closeDialogIssueSolusi"
            >
              Close
            </v-btn>
            <v-btn
              color="black"
              @click="saveIssueSolusi"
            >
              Save
            </v-btn>
          </v-card-actions>
      </v-card>
    </v-dialog>
    <v-bottom-sheet v-model="sheetNew">
      <v-card :style="{padding:'20px'}">
        <v-form :style="{display:'flex','margin-bottom':'10px'}" ref="formIssueBaru" v-model="validIssueBaru" lazy-validation>
          <v-text-field
            color="black"
            :counter="100"
            label="Tambahkan Issue Baru"
            v-model="issueBaru"
            :rules="issueBaruRules"
          ></v-text-field>
          <v-select
            color="black"
            style="margin-left: 20px;"
            v-model="warnSelected"
            :items="warnLevel"
            :rules="issueBaruRules"
            label="Warn Level"
            single-line
          ></v-select>
          <v-btn :style="{'margin-top':'10px'}" :disabled="!validIssueBaru" @click="addNewIssue" light color="white">ADD</v-btn>
        </v-form>
        <v-form :style="{display:'flex','margin-bottom':'10px'}" ref="formSolusiBaru" v-model="validSolusiBaru" lazy-validation>
          <v-text-field
            color="black"
            :counter="100"
            label="Solusi"
            v-model="solusiBaru"
            :rules="issueBaruRules"
          ></v-text-field>
          <v-text-field
            color="black"
            style="margin-left: 20px;"
            :counter="10"
            label="Harga"
            v-model="hargaSolusiBaru"
            :rules="hargaBaruRules"
            type="number"
          ></v-text-field>
          <v-btn :disabled="!validSolusiBaru" @click="addNewSolusi" light color="white">ADD</v-btn>
        </v-form>
        <v-form :style="{display:'flex','margin-bottom':'10px'}" ref="formBiayaTambahanBaru" v-model="validBiayaTambahanBaru" lazy-validation>
          <v-text-field
            color="black"
            :counter="100"
            label="Nama Biaya Tambahan"
            v-model="biayaTambahanBaru"
            :rules="issueBaruRules"
          ></v-text-field>
          <v-text-field
            color="black"
            style="margin-left: 20px;"
            :counter="10"
            label="Harga"
            v-model="hargaBiayaTambahanBaru"
            :rules="hargaBaruRules"
            type="number"
          ></v-text-field>
          <v-btn :disabled="!validBiayaTambahanBaru" @click="addNewBiayaTambahan" light color="white">ADD</v-btn>
        </v-form>
      </v-card>
    </v-bottom-sheet>
    <v-layout align-start justify-center row fill-height>
      <v-dialog scrollable dark v-if="dialogPC" persistent v-model="dialogPC" width="900">
        <v-card>
          <v-card-title class="headline grey darken-2" primary-title>
            {{this.idKomputer}}

          </v-card-title>
          <v-card-text style="height: 80vh;overflow: auto;" v-if="detailKomputer[0]">
            <h3 :style="{'text-align':'center'}">{{detailKomputer[0].namaKomputer}}</h3>
            <v-divider></v-divider>
            <p :style="{'margin-top':'10px'}">Nama User : {{detailKomputer[0].namaMasuk}}</p>
            <p>NoHP : {{detailKomputer[0].noHPMasuk}}</p>
            <v-divider></v-divider>
            <p :style="{'margin-top':'10px'}">Admin : {{detailKomputer[0].namaAdmin}}</p>
            <p>Tanggal Masuk : {{detailKomputer[0].tanggalMasuk}}</p>
            <v-divider></v-divider>
            <v-data-table
              light
              v-if="detailKomputer[0].keluhan[0 ]"
              :items="detailKomputer[0].keluhan"
              class="elevation-1"
              :headers="[{
                text: 'Keluhan',
                align: 'center',
                sortable: false,
                value: 'name'
              }]"
              hide-actions
            >
              <template slot="items" slot-scope="props">
                <td>{{ props.item.keluhan }}</td>
              </template>
            </v-data-table>
            <v-divider></v-divider>
            <v-data-table
              light
              v-if="detailKomputer[0].kelengkapan[0]"
              :items="detailKomputer[0].kelengkapan"
              class="elevation-1"
              :headers="[{
                text: 'Kelengkapan',
                align: 'center',
                sortable: false,
                value: 'name'
              }]"
              hide-actions
            >
              <template slot="items" slot-scope="props">
                <td>{{ props.item.kelengkapan }}</td>
              </template>
            </v-data-table>
            <v-divider style="margin-top: 20px;"></v-divider>
            <p v-if="typeDetail == 'teknisi'" style="margin-top: 10px;">Issue dan Solusi:</p>
            <p v-if="detailKomputer[1].tanggalValidasi || !detailKomputer[1].tanggalValidasi == ''">Terakhir Diubah User : {{detailKomputer[1].tanggalValidasi}}</p> 
            <v-card style="margin-top: 10px;" color="grey darken-4" v-for="issue in detailKomputer[1].issueSolusi">
              <v-card-title>
                {{issue.issue}}
                <v-chip v-if="issue.warnLevel == 0">
                  Warn : Low
                </v-chip>
                <v-chip v-if="issue.warnLevel == 1">
                  Warn : Mid
                </v-chip>
                <v-chip v-if="issue.warnLevel == 2">
                  Warn : High
                </v-chip>
              </v-card-title>
              <v-card-text>
                <v-card style="margin-top: 2px;" v-for="solusi in issue.solusi">
                  <v-card-title>
                    {{solusi.solusi}}
                  </v-card-title>
                  <v-card-text style="padding-bottom: 5px; padding-top: 0px;">
                    <v-chip>
                      <v-avatar color="grey darken-3" style="color: white;">
                        Rp
                      </v-avatar>
                        {{solusi.harga}}
                    </v-chip>
                    <v-chip v-if="solusi.choosen == 1">
                      <v-avatar color="grey darken-3" style="color: white;">
                        <v-icon>check_circle</v-icon>
                      </v-avatar>
                      Choosen
                    </v-chip>
                  </v-card-text>
                </v-card>
              </v-card-text>
            </v-card>
            <v-card v-if="detailKomputer[1].catatanTeknisi">
              <v-card-title>Catatan</v-card-title>
              <v-card-text>{{detailKomputer[1].catatanTeknisi}}</v-card-text>
            </v-card>
            <!-- disini -->
            <v-text-field
              v-if="!detailKomputer[1].catatanTeknisi && !detailKomputer[1].tanggalValidasi && typeDetail == 'teknisi'"
              :counter="200"
              label="Tambahkan Catatan"
              color="white"
              v-model="catatanTeknisi"
            ></v-text-field>
            <v-autocomplete
              v-model="selectedChoosenSolusi"
              :items="detailKomputer[1].choosenSolusi"
              v-if="detailKomputer[1].tanggalValidasi && detailKomputer[1].tanggalValidasi != 'no record' && !detailKomputer[2].tanggalDikerjakan && detailKomputer[1].choosenSolusi.length != 0"
              box
              chips
              color="grey lighten-2"
              label="Pilih Solusi Yang Akan Dikerjakan"
              item-text="solusi"
              item-value="idSolusi"
              multiple
            >
              <template
                slot="selection"
                slot-scope="data"
              >
                <v-chip>
                {{ data.item.solusi }}, RP. {{ data.item.harga }}
                </v-chip>
              </template>
              <template
                slot="item"
                slot-scope="data"
              >
                <template v-if="typeof data.item !== 'object'">
                  <v-list-tile-content v-text="data.item"></v-list-tile-content>
                </template>
                <template v-else>
                  <v-list-tile-content>
                    {{ data.item.solusi }}
                    <v-chip>
                    Harga : RP.{{ data.item.harga }}
                  </v-chip>
                  </v-list-tile-content>
                </template>
              </template>
            </v-autocomplete>
            <v-divider style="margin-top: 20px;"></v-divider>
            <p v-if="detailKomputer[2].tanggalDikerjakan && detailKomputer[3].tanggalSelesai != 'cancel'" >Dikerjakan : {{detailKomputer[2].tanggalDikerjakan}}</p>
            <p v-if="detailKomputer[2].tanggalDikerjakan && detailKomputer[3].tanggalSelesai != 'cancel'" >Solusi Dikerjakan : </p>
            <v-card color="grey lighten-2" style="margin-top: 5px;" light v-for="solusi in detailKomputer[2].solusiDikerjakan">
              <v-card-title>{{solusi.solusi}}</v-card-title>
              <v-card-text style="padding-bottom: 5px; padding-top: 0px;">
                <v-chip color="grey lighten-1">
                  <v-avatar color="grey darken-3" style="color: white;">
                    Rp
                  </v-avatar>
                    {{solusi.harga}}
                </v-chip>
              </v-card-text>
            </v-card>
            <v-autocomplete
              v-model="selectedBiayaTambahan"
              :items="biayaTambahan"
              v-if="detailKomputer[2].tanggalDikerjakan && !detailKomputer[3].tanggalSelesai"
              box
              chips
              color="grey lighten-2"
              label="Tambahkan Biaya Tambahan"
              item-text="tambahan"
              item-value="idTambahan"
              @focus="fetchBiayaTambahan"
              multiple
            >
              <template
                slot="selection"
                slot-scope="data"
              >
                <v-chip>
                {{ data.item.tambahan }}, RP. {{ data.item.harga }}
                </v-chip>
              </template>
              <template
                slot="item"
                slot-scope="data"
              >
                <template v-if="typeof data.item !== 'object'">
                  <v-list-tile-content v-text="data.item"></v-list-tile-content>
                </template>
                <template v-else>
                  <v-list-tile-content color="black">
                    {{ data.item.tambahan }}
                    <v-chip>
                    Harga : RP.{{ data.item.harga }}
                  </v-chip>
                  </v-list-tile-content>
                </template>
              </template>
            </v-autocomplete>
            <v-card light style="margin-top: 20px;" v-if="detailKomputer[3].tanggalSelesai && detailKomputer[3].tanggalSelesai != 'cancel'">
              <v-card-title>Selesai</v-card-title>
              <v-card-text>{{detailKomputer[3].tanggalSelesai}}</v-card-text>
            </v-card>
            <v-card light style="margin-top: 20px;" v-if="detailKomputer[3].tanggalSelesai && detailKomputer[3].tanggalSelesai == 'cancel'">
              <v-card-title>Dibatalkan</v-card-title>
              <v-card-text>{{detailKomputer[1].tanggalValidasi}}</v-card-text>
            </v-card>
            <div v-if="typeDetail == 'teknisi'">
              <v-btn
                v-if="!detailKomputer[1].tanggalValidasi || detailKomputer[1].tanggalValidasi == ''"
                color="black"
                @click="openDialogIssueSolusi"
              >
                <v-icon style="margin-right: 10px;">add</v-icon>
                Tambahkan Issue Dan Solusi
              </v-btn>
              <v-btn
                v-if="(!detailKomputer[1].tanggalValidasi || detailKomputer[1].tanggalValidasi == '') && detailKomputer[1].issueSolusi[0]"
                color="black"
                @click="validasiIssueSolusi"
              >
                <v-icon style="margin-right: 10px;">check_circle</v-icon>
                Validasi
              </v-btn>
              <v-btn
                v-if="detailKomputer[1].tanggalValidasi && detailKomputer[1].tanggalValidasi != 'no record' && !detailKomputer[2].tanggalDikerjakan 
                && detailKomputer[1].choosenSolusi.length != 0"
                color="black"
                @click="mulaiMengerjakan"
              >
                <v-icon style="margin-right: 10px;">check_circle</v-icon>
                Mulai Mengerjakan
              </v-btn>
              <v-btn
                v-if="detailKomputer[2].tanggalDikerjakan && !detailKomputer[3].tanggalSelesai"
                color="black"
                @click="selesaiMengerjakan"
              >
                <v-icon style="margin-right: 10px;">check_circle</v-icon>
                Selesai Mengerjakan
              </v-btn>
            </div>
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn v-if="typeDetail == 'teknisi'" @click="openSheetNew"><v-icon style="margin-right: 10px;">add</v-icon>Tambahkan /Issue /Solusi /Biaya tambahan baru</v-btn>
            <v-btn
              light
              color="white"
              @click="detailsAllClose"
            >
              Close
            </v-btn>
            <v-btn
              v-if="typeDetail == 'all'"
              color="black"
              @click="kerjakan"
            >
              Kerjakan
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-card v-if="loading == false" dark max-width="2000" width="1500" :style="{height:'92vh'}">
        <v-card-title :style="{display:'flex', 'align-items':'flex-start', 'justify-content':'center'}" primary-title>
          <v-text-field v-model="searchValueAll" prepend-icon="search" :style="{width:'100%'}" label="Search Komputer" color="white"></v-text-field>
        </v-card-title>
        <v-card-text :style="{height:'80%',overflow:'auto',display:'flex', 'align-items':'flex-start', 'justify-content':'center', 'flex-wrap':'wrap'}">
            <v-card v-for="(computer,index) in filteredAll" v-if="computer.tanggalDikerjakan == null || computer.tanggalDikerjakan == '' || computer.idTeknisi == null" :key="index" light :style="{margin:'10px',width:'340px',display:'flex', 'align-items':'flex-start', 'justify-content':'center', 'flex-wrap':'wrap'}">
              <v-card-title>{{computer.idKomputer}}</v-card-title>
              <v-card-text v-if="computer.tanggalDikerjakan != 'cancel'" :style="{display:'flex', 'align-items':'flex-start', 'justify-content':'center', 'flex-wrap':'wrap'}">
                <v-chip :style="{width:'95%'}" label color="grey darken-3" text-color="white">
                    <v-icon left>label</v-icon>Masuk : {{computer.tanggalMasuk}}
                </v-chip>
              </v-card-text>
              <v-card-text v-if="computer.tanggalDikerjakan == 'cancel'" :style="{'text-align':'left'}">
                  <v-chip :style="{width:'95%'}" label color="grey darken-1" text-color="white">
                      <v-icon left>label</v-icon>Masuk : {{computer.tanggalMasuk}}
                  </v-chip>
                  <v-chip v-if="(computer.tanggalValidasi != null)" :style="{width:'95%'}" label color="black" text-color="white">
                      <v-icon left>cancel</v-icon>Dibatalkan : {{computer.tanggalValidasi}}
                  </v-chip>
                  <v-chip v-if="(computer.tanggalKeluar != null && computer.tanggalDikerjakan != '')" :style="{width:'95%'}" label color="grey darken-4" text-color="white">
                    <v-icon left>shopping_cart</v-icon>Diambil : {{computer.tanggalKeluar}}
                </v-chip>
              </v-card-text>
              <v-btn @click="detailsAllOpen(computer.idKomputer, 'all')" :style="{'margin-bottom':'20px'}" dark color="grey darken-4">
                View Details
                <v-icon right dark>search</v-icon>
              </v-btn>
            </v-card>
        </v-card-text>
      </v-card>
      <v-card width="900" :style="{height:'92vh'}">
        <v-card-title :style="{display:'flex', 'align-items':'flex-start', 'justify-content':'center'}" primary-title>
          <v-text-field v-model="searchValueTeknisi" prepend-icon="search" :style="{width:'100%'}" label="Komputer Dikerjakan" color="black"></v-text-field>
          <v-select
            color="black"
            :items="categoryFilter"
            v-model="selectedCategoryFilter"
            label="Filter Search"
            outline
          ></v-select>
        </v-card-title>
        <v-card-text :style="{height:'70%',overflow:'auto',display:'flex', 'align-items':'flex-start', 'justify-content':'center', 'flex-wrap':'wrap'}">
            <v-card v-for="(computer,index) in filteredTeknisi" dark :style="{margin:'10px',width:'340px',display:'flex', 'align-items':'flex-start', 'justify-content':'center', 'flex-wrap':'wrap'}">
              <v-card-title>{{computer.idKomputer}}</v-card-title>
              <v-card-text v-if="computer.tanggalDikerjakan != 'cancel'" :style="{display:'flex', 'align-items':'flex-start', 'justify-content':'center', 'flex-wrap':'wrap'}">
                <v-chip :style="{width:'95%'}" label color="grey lighten-1" text-color="white">
                    <v-icon left>label</v-icon>Masuk : {{computer.tanggalMasuk}}
                </v-chip>
                <v-chip v-if="(computer.tanggalValidasi != null && computer.tanggalValidasi != '')" :style="{width:'95%'}" label color="grey darken-1" text-color="white">
                    <v-icon left>assignment_turned_in</v-icon>Validasi : {{computer.tanggalValidasi}}
                </v-chip>
                <v-chip v-if="(computer.tanggalDikerjakan != null && computer.tanggalDikerjakan != '')" :style="{width:'95%'}" label color="grey darken-2" text-color="white">
                    <v-icon left>build</v-icon>Dikerjakan : {{computer.tanggalDikerjakan}}
                </v-chip>
                <v-chip v-if="(computer.tanggalSelesai != null && computer.tanggalDikerjakan != '')" :style="{width:'95%'}" label color="grey darken-4" text-color="white">
                    <v-icon left>check_circle</v-icon>Selesai : {{computer.tanggalSelesai}}
                </v-chip>
                <v-chip v-if="(computer.tanggalKeluar != null && computer.tanggalDikerjakan != '')" :style="{width:'95%'}" label color="grey darken-4" text-color="white">
                    <v-icon left>shopping_cart</v-icon>Diambil : {{computer.tanggalKeluar}}
                </v-chip>
              </v-card-text>
              <v-card-text v-if="computer.tanggalDikerjakan == 'cancel'" :style="{'text-align':'left'}">
                  <v-chip :style="{width:'95%'}" label color="grey darken-1" text-color="white">
                      <v-icon left>label</v-icon>Masuk : {{computer.tanggalMasuk}}
                  </v-chip>
                  <v-chip v-if="(computer.tanggalValidasi != null)" :style="{width:'95%'}" label color="black" text-color="white">
                      <v-icon left>cancel</v-icon>Dibatalkan : {{computer.tanggalValidasi}}
                  </v-chip>
                  <v-chip v-if="(computer.tanggalKeluar != null && computer.tanggalDikerjakan != '')" :style="{width:'95%'}" label color="grey darken-4" text-color="white">
                    <v-icon left>shopping_cart</v-icon>Diambil : {{computer.tanggalKeluar}}
                </v-chip>
              </v-card-text>
              <v-btn @click="detailsAllOpen(computer.idKomputer, 'teknisi')" :style="{'margin-bottom':'20px'}" dark color="grey darken-4">
                View Details
                <v-icon right dark>search</v-icon>
              </v-btn>
            </v-card>
        </v-card-text>
      </v-card>
    </v-layout>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import config from '../config.js'

  export default {
    name: 'Hello',
    data: () => ({
      selectedCategoryFilter:'',
      categoryFilter:['All','Masuk','Validasi','Dikerjakan','Dibatalkan','Selesai'],
      catatanTeknisi:'',
      selectedBiayaTambahan:[],
      biayaTambahan:[],
      selectedChoosenSolusi:[],
      issueSolusiKomputer:[],
      solusi:[],
      selectedSolusi:[],
      issue:[],
      selectedIssue:null,
      dialogIssueSolusi:false,
      warnSelected:'',
      warnLevel:['Low','Mid','High'],
      issueSolusiDipilih:[{
        idIssue:1,
        solusi:[1,2,3]
      }],
      sheetNew: false,
      validIssueBaru: true,
      validSolusiBaru: true,
      validBiayaTambahanBaru: true,
      biayaTambahanBaru: '',
      hargaBiayaTambahanBaru: null,
      solusiBaru: '',
      hargaSolusiBaru: null,
      issueBaru: '',
      issueBaruRules: [
        v => !!v || 'Field required',
        v => (v && v.length <= 100) || 'MNst be less than 100 characters'
      ],
      hargaBaruRules: [
        v => !!v || 'Field required',
        v => (v && v.length <= 10) || 'MNst be less than 10 characters'
      ],
      typeDetail:'',
      idKomputer:'',
      detailKomputer:[],
      dialogPC:false,
      searchBy:'all',
      selectedCategory:'Semua',
      category:['Semua','Validasi','Dikerjakan','Dibatalkan','Selesai'],
      searchValueAll:'',
      computersAll:[],
      filteredAll:[],
      searchValueTeknisi:'',
      computersTeknisi:[],
      filteredTeknisi:[],
      loading:true
    }),
    created(){
      this.getSemuaKomputer()
      if(localStorage.getItem('@idKomputer') && localStorage.getItem('@dialogOpenAll')){
        if(localStorage.getItem('@typeDetail')){
          this.typeDetail = localStorage.getItem('@typeDetail')
        }
        this.dialogPC = true
        this.idKomputer = localStorage.getItem('@idKomputer')
        this.getKomputerDetail(this.idKomputer)
        .then(result =>{
          this.detailKomputer = result.data
          console.log(this.detailKomputer)
          this.dialogPC = true
        })
      }
    },
    methods:{
      getCurrentDate(){
        let currentdate = new Date(); 
        return currentdate.getDate() + "/"
            + (currentdate.getMonth()+1)  + "/" 
            + currentdate.getFullYear() + " @ "  
            + currentdate.getHours() + ":"  
            + currentdate.getMinutes() + ":" 
            + currentdate.getSeconds()
      },
      fetchBiayaTambahan(){
        axios.get(`http://${config.ip}:2000/allBiayaTambahan`)
        .then(result =>{
          this.biayaTambahan = result.data
        })
      },
      selesaiMengerjakan(){
        console.log(this.selectedBiayaTambahan)
        console.log(this.idKomputer)
        this.selectedBiayaTambahan.forEach(v =>{
          axios.post(`http://${config.ip}:2000/biayaTambahanKomputer`,{
            idKomputer : this.idKomputer,
            idTambahan : v
          })
          .then(result =>{
            
          })
        })
        axios.post(`http://${config.ip}:2000/setTanggalSelesai`,{
          idKomputer : this.idKomputer,
          tanggalSelesai : this.getCurrentDate()
        })
        .then(result =>{
          console.log(result)
        })
        alert('Selamat anda telah selesai')
        this.getKomputerDetail(this.idKomputer)
        .then(result =>{
          this.detailKomputer = result.data
          console.log(this.detailKomputer)
          this.dialogPC = true
          this.getSemuaKomputer()
        })
        .then(()=>{
          axios.post(`http://${config.ip}:2000/sms`,{
            number: this.detailKomputer[0].noHPMasuk,
            msg: `System Computer Service Centre : ${this.detailKomputer[0].namaMasuk} Komputer anda telah selesai, Silahkan cek biaya service di website kami kemudian ambil komputer anda di toko kami, Terimakasih telah menggunakan layanan kami #TEAM_AI`
          })
          .then(result =>{
            console.log('hasil')
            console.log(result)
          })
        })
      },
      mulaiMengerjakan(){
        console.log(this.selectedChoosenSolusi)
        axios.post(`http://${config.ip}:2000/setTanggalDikerjakan`,{
          idKomputer : this.idKomputer,
          tanggalDikerjakan : this.getCurrentDate()
        })
        .then(result =>{
          console.log(result)
        })
        this.selectedChoosenSolusi.forEach(v =>{
          axios.post(`http://${config.ip}:2000/solusiDikerjakan`,{
            idKomputer : this.idKomputer,
            idSolusi : v
          })
          .then(result =>{
            console.log(result)
          })
        })
        alert('Selamat Mengerjakan')
        this.getKomputerDetail(this.idKomputer)
        .then(result =>{
          this.detailKomputer = result.data
          console.log(this.detailKomputer)
          this.dialogPC = true
          this.getSemuaKomputer()
          axios.post(`http://${config.ip}:2000/sms`,{
            number: this.detailKomputer[0].noHPMasuk,
            msg: `System Computer Service Centre : ${this.detailKomputer[0].namaMasuk} Komputer anda dalam proses validasi, Silahkan cek dan lakukan validasi komputer anda di website kami`
          })
          .then(result =>{
            console.log('hasil')
            console.log(result)
          })
        })
      },
      validasiIssueSolusi(){
        //catatanteknisi
        axios.post(`http://${config.ip}:2000/setTanggalValidasi`,{
          idKomputer : this.idKomputer,
          tanggalValidasi : 'no record'
        }).then(result =>{
          axios.post(`http://${config.ip}:2000/setCatatanTeknisi`,{
            idKomputer : this.idKomputer,
            catatan : this.catatanTeknisi
          }).then(result =>{
            this.dialogPC = true
            this.idKomputer = localStorage.getItem('@idKomputer')
            this.getKomputerDetail(this.idKomputer)
            .then(result =>{
              this.detailKomputer = result.data
              console.log(this.detailKomputer)
              this.dialogPC = true
              this.getSemuaKomputer()
              alert('Silahkan Tunggu Hasil Validasi User')
            })
          })
        })
      },
      saveIssueSolusi(){
        console.log(this.selectedIssue)
        console.log(this.selectedSolusi)
        if(this.selectedIssue != null && this.selectedSolusi.length != 0){
          this.selectedSolusi.forEach(v =>{
            axios.post(`http://${config.ip}:2000/addPermasalahanKomputer`,{
              idKomputer : this.idKomputer,
              idIssue : this.selectedIssue,
              idSolusi : v
            })
            .then(result =>{
              console.log(result)
            })
          })
          this.closeDialogIssueSolusi()
          this.dialogPC = true
          this.idKomputer = localStorage.getItem('@idKomputer')
          this.getKomputerDetail(this.idKomputer)
          .then(result =>{
            this.detailKomputer = result.data
            console.log(this.detailKomputer)
            this.dialogPC = true
          })
          //here
        }else{
          alert('Data Belum Terisi')
        }
        console.log(this.detailKomputer)
      },
      fetchSolusi(){
        axios.get(`http://${config.ip}:2000/allSolusi`)
        .then(result =>{
          this.solusi = result.data
          console.log(this.selectedSolusi)
        })
      },
      fetchIssue(){
        axios.get(`http://${config.ip}:2000/allIssue`)
        .then(result =>{
          let temp = []
          result.data.forEach((v,i) =>{
            temp.push(v)
            if(v.warnLevel == 0){
              temp[i].warnText = 'Low'
            }else if(v.warnLevel == 1){
              temp[i].warnText = 'Mid'
            }else if(v.warnLevel == 2){
              temp[i].warnText = 'High'
            }
          })
          this.issue = temp
          console.log(this.selectedIssue)
        })
      },
      openDialogIssueSolusi(){
        this.dialogIssueSolusi = true
      },
      closeDialogIssueSolusi(){
        this.dialogIssueSolusi = false
      },
      logout(){
        this.idTeknisi = ''
        localStorage.removeItem('@idTeknisi')
        this.$emit('logout')
      },
      kerjakan(){
        axios.post(`http://${config.ip}:2000/kerjakan`,{
          idKomputer : localStorage.getItem('@idKomputer'),
          idTeknisi : localStorage.getItem('@idTeknisi')
        })
        .then(result =>{
          if(result.data == 'succes'){
            alert('Request Succes')
            localStorage.removeItem('@idKomputer')
            this.getSemuaKomputer()
            this.dialogPC = false
          }
        })
      },
      openSheetNew(){
        this.sheetNew = true
      },
      closeSheetNew(){
        this.sheetNew = false 
      },
      addNewBiayaTambahan(){
        console.log('gasike')
        if (this.$refs.formBiayaTambahanBaru.validate()) {
          axios.post(`http://${config.ip}:2000/addBiayaTambahan`,{
            tambahan : this.biayaTambahanBaru,
            harga : this.hargaBiayaTambahanBaru
          })
          .then(result =>{
            if(result.data == 'gagal'){
              alert('Data sudah ada')
            }else{
              alert('Data berhasil ditambahkan')
            }
          })
          .catch(err =>{
            alert('Gagal')
          })
        }
      },
      addNewIssue(){
        if (this.$refs.formIssueBaru.validate()) {
          let warnNumber = 0
          switch(this.warnSelected){
            case 'Low':{
              warnNumber = 0
              break
            }
            case 'Mid':{
              warnNumber = 1
              break
            }
            case 'High':{
              warnNumber = 2
              break 
            }
          }
          axios.post(`http://${config.ip}:2000/addIssue`,{
            issue : this.issueBaru,
            warnLevel : warnNumber
          })
          .then(result =>{
            if(result.data == 'gagal'){
              alert('Data sudah ada')
            }else{
              alert('Data berhasil ditambahkan')
            }
          })
          .catch(err =>{
            alert('Gagal')
          })
        }
        
      },
      addNewSolusi(){
        if (this.$refs.formSolusiBaru.validate()) {
          axios.post(`http://${config.ip}:2000/addSolusi`,{
            solusi : this.solusiBaru,
            harga : this.hargaSolusiBaru
          })
          .then(result =>{
            if(result.data == 'gagal'){
              alert('Data sudah ada')
            }else{
              alert('Data berhasil ditambahkan')
            }
          })
          .catch(err =>{
            alert('Gagal')
          })
        }
      },
      detailsAllOpen(idKomputer, type){
        console.log(idKomputer)
        this.idKomputer = idKomputer
        localStorage.setItem('@idKomputer', idKomputer)
        localStorage.setItem('@dialogOpenAll', true)
        this.getKomputerDetail(this.idKomputer)
        .then(result =>{
          this.detailKomputer = result.data
          console.log(this.detailKomputer)
          this.dialogPC = true
        })
        if(type == 'all'){
          localStorage.setItem('@typeDetail', 'all')
          this.typeDetail = 'all'
        }else{
          localStorage.setItem('@typeDetail', 'teknisi')
          this.typeDetail = 'teknisi'
        }
        
      },
      detailsAllClose(){
        localStorage.removeItem('@idKomputer')
        localStorage.removeItem('@dialogOpenAll')
        localStorage.removeItem('@typeDetail')
        this.dialogPC = false
      },
      getKomputerDetail(idKomputer){
        return axios.post(`http://${config.ip}:2000/komputerDetail`,{
          idKomputer : idKomputer
        })
      },
      getSemuaKomputer(){
        this.computersAll = []
        this.computersTeknisi = []
        axios.get(`http://${config.ip}:2000/semuaKomputer`)
        .then(result =>{
          this.computersAll = result.data
          console.log(this.computersAll)
        })
        .then(()=>{
          let temp = []
          this.computersAll.forEach(v =>{
            if(v.idTeknisi == localStorage.getItem('@idTeknisi')){
              this.computersTeknisi.push(v)
            }else{
              temp.push(v)
            }
          })
          let tempAll = []
          temp.forEach(v =>{
            if(!v.idTeknisi){
              tempAll.push(v)
            }
          })
          this.computersAll = tempAll
          this.filteredAll = this.computersAll
          this.filteredTeknisi = this.computersTeknisi
          this.loading = false
          console.log(this.computersTeknisi)
        })
      },
    },
    watch:{
      searchValueAll:function(newValue){
        this.filteredAll = this.computersAll.filter(v => v.idKomputer.match(newValue))
      },
      searchValueTeknisi:function(newValue){
        console.log(this.selectedCategoryFilter)
        switch(this.selectedCategoryFilter){
          case 'All':{
            this.filteredTeknisi = this.computersTeknisi.filter(v => (v.idKomputer.match(newValue)))
            break;
          }
          case 'Masuk':{
            this.filteredTeknisi = this.computersTeknisi.filter(v => (v.idKomputer.match(newValue) && v.tanggalValidasi == null))
            break;
          }
          case 'Validasi':{
            this.filteredTeknisi = this.computersTeknisi.filter(v =>  (v.idKomputer.match(newValue) && v.tanggalValidasi != null && v.tanggalDikerjakan == null && v.tanggalDikerjakan != 'cencel'))
            break;
          }
          case 'Dikerjakan':{
            this.filteredTeknisi = this.computersTeknisi.filter(v =>  (v.idKomputer.match(newValue) && v.tanggalDikerjakan != null && v.tanggalSelesai == null && v.tanggalDikerjakan != 'cencel'))
            break;
          }
          case 'Selesai':{
            this.filteredTeknisi = this.computersTeknisi.filter(v =>  (v.idKomputer.match(newValue) && v.tanggalSelesai != null && v.tanggalDikerjakan != 'cancel'))
            break;
          }
          case 'Dibatalkan':{
            this.filteredTeknisi = this.computersTeknisi.filter(v =>  (v.idKomputer.match(newValue) && v.tanggalDikerjakan == 'cancel'))
            break;
          }
        }
        
      },
      selectedCategoryFilter: function(newValue){
        
        if(newValue == 'All'){
          this.filteredTeknisi = this.computersTeknisi
        }else if(newValue == 'Masuk'){
          this.filteredTeknisi = this.computersTeknisi.filter(v => (v.tanggalValidasi == null))
        }else if(newValue == 'Validasi'){
          this.filteredTeknisi = this.computersTeknisi.filter(v => (v.tanggalValidasi != null && v.tanggalDikerjakan == null && v.tanggalDikerjakan != 'cencel'))
        }else if(newValue == 'Dikerjakan'){
          this.filteredTeknisi = this.computersTeknisi.filter(v => (v.tanggalDikerjakan != null && v.tanggalSelesai == null && v.tanggalDikerjakan != 'cencel'))
        }else if(newValue == 'Selesai'){
          this.filteredTeknisi = this.computersTeknisi.filter(v => (v.tanggalSelesai != null && v.tanggalDikerjakan != 'cancel'))
        }else if(newValue == 'Dibatalkan'){
          this.filteredTeknisi = this.computersTeknisi.filter(v => (v.tanggalDikerjakan == 'cancel'))
        }
      }
    }
  }
</script>

<style>

</style>
