<template>
<div class="container py-2">
  <!-- To taki generator nowych wykresow, nieskonczony na razie. 
  1. wybierasz typ jaki chcesz (line,bar)
  2. wybierasz baze z dostepnych (na razie nie ma ich)
  3. wybierasz keys po ktorym chcesz pobrac dane i wybierasz co chcesz zrobic z tym np suma, srednia
  4. po wygenerowaniu przekierowuje cie do /chart/:id/:type
   -->


  <div v-if="steps.database" class="columns">
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
            <router-link to="/"><button class="btn btn-primary"> Cofnij </button> </router-link>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <br>
      <h1 class="text-dark">Wybierz bazę danych</h1>
      <br>
      <br>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-6 col-lg-6 col-xl-6 col-6">
      <h4>Użyj gotowej bazy danych</h4>
      <div class="accordion">
        <input type="checkbox" id="accordion-population" name="accordion-checkbox" checked hidden>
        <label class="accordion-header" for="accordion-population">
          <i class="icon icon-arrow-right mr-1"></i>
          Populacja
        </label>
        <div class="accordion-body">
          <ul>
            <li>
              <p style="color: gray;">Pozwala zwirtualizować dane na temat populacji ludzkiej oraz wybrać wiek, rok,
                płeć oraz kraj badanej grupy</p>
              <button @click="steps.database = false; steps.type = true; steps.keysClicked = true" class="btn btn-primary">Użyj bazy</button>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-6 col-lg-6 col-xl-6 col-6">
      <h4>Stwórz własne dane</h4>
      <p style="color: gray;">Za pomocą prostego interfesjsu dodaj liczbę danych, ich rodzaj oraz warstwy, które chcesz
        umieścic na wykresie</p>
      <button @click="steps.database = false; steps.type = true; steps.keysOwnClicked = true" class="btn btn-primary">Stwórz nowe dane</button>
    </div>

  </div>

  
  <div v-else-if="steps.type" class="columns">

    <div v-if="steps.keysClicked == true" class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">
          <button @click="resetData(); steps.type = false; steps.database = true; steps.keysClicked = false" class="btn btn-primary"> Cofnij </button>
      <button @click="steps.type = false; steps.keys = true" class="btn btn-primary"> Dalej </button>
    </div>
    <div v-else-if="steps.keysOwnClicked == true" class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">
          <button @click="resetData(); steps.type = false; steps.database = true; steps.keysOwnClicked = false" class="btn btn-primary"> Cofnij </button>
      <button @click="steps.type = false; steps.keysOwn = true" class="btn btn-primary"> Dalej </button>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <h2>Wybierz typ wykresu</h2>
    </div>
    <transition-group :name="'shuffleMedium'" tag="div" class="deck">
      <div v-for="(type, i) in types" :key="type.id" class="type" @click="setFirst(i)">
        <h5 :key="type.title">{{type.human}}</h5>
        <img class="image" :src="type.image" :alt="type.title" :key="type.image">
      </div>
    </transition-group>


  </div>

  <div v-else-if="steps.keys" class="columns">
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">
      <button @click="steps.type = true; steps.keys = false" class="btn btn-primary"> Cofnij </button>
      <button v-if="isValid" @click="useData(); steps.keys = false; steps.customize = true"
        class="btn btn-primary">Dalej</button>
      <div style="cursor: pointer;" @click="checkIsValid()" v-else><button
          class="btn btn-primary disabled">Dalej</button>
      </div>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <h2>Wybierz dane, które chcesz zwizualizować</h2>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-6 col-lg-6 col-xl-6 col-6">
      <div class="accordion">
        <input type="checkbox" id="accordion-1" name="accordion-checkbox" checked hidden>
        <label class="accordion-header" for="accordion-1">
          <i class="icon icon-arrow-right mr-1"></i>
          Dodaj pojedyncze dane
        </label>
        <div class="accordion-body">
          <ul>
            <li>Wybierz wiek: <br>
              <input v-model="age" style="width: 10em;" class="slider tooltip" type="range" min="0" max="99" value="50"
                oninput="this.setAttribute('value', this.value);">
            </li>
          </ul>
          <ul>
            <li>

              <div class="accordion">
                <input type="checkbox" id="accordion-2" name="accordion-checkbox" hidden>
                <label class="accordion-header" for="accordion-2">
                  <i class="icon icon-arrow-right mr-1"></i>
                  Wybierz kraj:
                </label>
                <div class="accordion-body">
                  <ul class="menu menu-nav">
                    <li class="menu-item" v-for="country in countries" :key="country.value"><a
                        v-bind:class="[countryValue == country.value ? 'country-checked' : '']"><label
                          @click="selectCountry(country.name, country.value)"><input
                            style="visibility: hidden; position:absolute;" name="country"
                            type="checkbox">{{country.name}}</label></a></li>
                  </ul>
                </div>
              </div>
            </li>
          </ul>
          <ul>
            <li>

              <div class="accordion">
                <input type="checkbox" id="accordion-3" name="accordion-checkbox" hidden>
                <label class="accordion-header" for="accordion-3">
                  <i class="icon icon-arrow-right mr-1"></i>
                  Wybierz płeć:
                </label>
                <div class="accordion-body">
                  <ul>
                    <div class="form-group">
                      <label class="form-radio">
                        <input type="radio" name="gender" value="male" v-model="gender" checked>
                        <i class="form-icon"></i> Mężczyźni
                      </label>
                      <label class="form-radio">
                        <input type="radio" v-model="gender" value="female" name="gender">
                        <i class="form-icon"></i> Kobiety
                      </label>
                      <label class="form-radio">
                        <input type="radio" v-model="gender" value="all" name="gender">
                        <i class="form-icon"></i> Obie
                      </label>
                    </div>
                  </ul>
                </div>
              </div>
            </li>
          </ul>
          <ul>
            <li>

              <div class="accordion">
                <input type="checkbox" id="accordion-4" name="accordion-checkbox" hidden>
                <label class="accordion-header" for="accordion-4">
                  <i class="icon icon-arrow-right mr-1"></i>
                  Wybierz rok:
                </label>
                <div class="accordion-body">
                  <ul>
                    <p style="color: gray;">Wybierz rok między 1950 a 2100</p>
                    <input v-model="date" style="width: 15em;" class="form-input" type="number" placeholder="00"
                      min="1950" value="1960" max="2100">
                  </ul>
                </div>
              </div>
            </li>
            <li><button @click="getDataApi()" class="btn btn-primary">Zaakceptuj</button></li>
          </ul>
        </div>
      </div>

    </div>

    <div class="column col-xs-12 col-sm-12 col-md-6 col-lg-6 col-xl-6 col-6">
      <div class="accordion">
        <input type="checkbox" id="accordion-5" name="accordion-checkbox" checked hidden>
        <label class="accordion-header" for="accordion-5">
          <i class="icon icon-arrow-right mr-1"></i>
          Wybrane dane do wirtualizacji
        </label>
        <div class="accordion-body">
          <ul v-if="chartDataArray.length == 0">
            <li>
              <p style="color: gray;">Nie dodałeś jeszcze żadnych danych</p>
            </li>
          </ul>
          <ul v-else v-for="(chart, index) in chartDataArray" :key="index">
            <li>
              <div class="accordion">
                <input type="checkbox" :id="'accordion'+index" name="accordion-checkbox" checked hidden>
                <label style="display: flex; align-items: center;" class="accordion-header" :for="'accordion'+index">
                  <i class="icon icon-arrow-right mr-1"></i>
                  Dane {{index+1}} <button @click="deleteData(chart)" class="ml-2 btn btn-sm btn-error">Usuń</button>
                </label>
                <div class="accordion-body">
                  <ul>
                    <li style="display: flex; flex-direction: column">
                      <span>Kraj: <b>{{chart.name}}</b></span>
                      <span>Płeć: <b>{{chart.gender}}</b></span>
                      <span>Rok: <b>{{chart.date}}</b></span>
                    </li>
                  </ul>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>



    <!-- 
 <ul v-else style="display: flex; flex-direction: column;">
                <li v-for="chart in chartDataArray">
    
        </li> 
            </ul> -->



  </div>


  <div v-else-if="steps.keysOwn" class="columns">
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">
      <button @click="steps.type = true; steps.keysOwn = false;" class="btn btn-primary"> Cofnij </button>
      <button v-if="isValidCollection == true && isValidDocument == true"
        @click="useOwnData(); steps.keysOwn = false; steps.customizeOwn = true" class="btn btn-primary">Dalej</button>
      <div style="cursor: pointer;" @click="checkIsValid()" v-else><button class="btn btn-primary disabled">Dalej</button>
      </div>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <h2>Dodaj dane, które chcesz zwizualizować</h2>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-6 col-lg-6 col-xl-6 col-6">
      <div class="accordion">
        <input type="checkbox" id="accordion-own1" name="accordion-checkbox" checked hidden>
        <label class="accordion-header" for="accordion-own1">
          <i class="icon icon-arrow-right mr-1"></i>
          Dodaj dane
        </label>
        <div class="accordion-body">
          <ul>
            <li>
              <input required="required" placeholder="Nazwa warstwy" v-model="collectionName" type="text">
              <button @click="collectionAdd()" class="btn btn-primary btn-sm">Dodaj warstwę</button>
            </li>
            <li v-for="(collectionData, index) in ownChartDataArray.collections" :key="index">
              <div class="accordion">
                <input type="checkbox" :id="'accordion-collection'+index" name="accordion-checkbox" checked hidden>
                <label style="display: flex; align-items: center;" class="accordion-header"
                  :for="'accordion-collection'+index">
                  <i class="icon icon-arrow-right mr-1"></i>
                  {{collectionData.collectionName}} <button @click="deleteCollection(index)"
                    class="ml-2 btn btn-sm btn-error">Usuń</button>
                </label>
                <div class="accordion-body">
                  <ul>
                    <li>
                      <input v-model.lazy="documentName" placeholder="nazwa" style="width:25%;"
                        type="text">
                      :
                      <input v-model="documentValue" placeholder="wartość" style="width:25%;" type="number">
                      <button @click="documentAdd(index)" class="btn btn-primary btn-sm">Dodaj informację</button>
                    </li>
                  </ul>

                  <ul>
                    <li style="display: flex; flex-direction: column">
                      <span style="margin: 5px 0;" v-for="(documentData, documentIndex) in collectionData.documents"
                        :key="documentIndex"> {{documentData.documentName}}: <b>{{documentData.documentValue}}</b>
                        <button @click="deleteDocument(index, documentIndex)"
                          class="ml-2 btn btn-sm btn-error">Usuń</button></span>
                    </li>
                  </ul>
                </div>
              </div>
            </li>
          </ul>


        </div>
      </div>

    </div>

    <div class="column col-xs-12 col-sm-12 col-md-6 col-lg-6 col-xl-6 col-6">
      <div class="accordion">
        <input type="checkbox" id="accordion-ownData" name="accordion-checkbox" checked hidden>
        <label class="accordion-header" for="accordion-ownData">
          <i class="icon icon-arrow-right mr-1"></i>
          Wybrane dane do wirtualizacji
        </label>
        <div class="accordion-body">
          <ul v-if="ownChartDataArray.collections.length == 0">
            <li>
              <p style="color: gray;">Nie dodałeś jeszcze żadnych danych</p>
            </li>
          </ul>
          <ul v-else v-for="(collection, index) in ownChartDataArray.collections" :key="index">
            <li>
              <div class="accordion">
                <input type="checkbox" :id="'accordion-ownData'+index" name="accordion-checkbox" checked hidden>
                <label style="display: flex; align-items: center;" class="accordion-header"
                  :for="'accordion-ownData'+index">
                  <i class="icon icon-arrow-right mr-1"></i>
                  {{collection.collectionName}} <button @click="deleteCollection(index)"
                    class="ml-2 btn btn-sm btn-error">Usuń</button>
                </label>
                <div class="accordion-body">
                  <ul>
                    <li style="display: flex; flex-direction: column">
                      <span v-for="(document, index) in collection.documents" :key="index">
                        {{document.documentName}}: <b>{{document.documentValue}}</b>
                      </span>
                    </li>
                  </ul>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>


  </div>



  <div v-else-if="steps.customize" class="columns">
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">


      
      <button @click="steps.keys = true; steps.customize = false" class="btn btn-primary"> Cofnij </button>
      <button @click="steps.customize = false; steps.finish = true" class="btn btn-primary"
        :disabled="hasColor ? false : true">Dalej </button>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <h2>Spersonalizuj wygląd wykresu</h2>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-6 col-xl-6 col-6">
      <ChartContainer :type="chart.type" :data="chart.data" :id="0" :options="chart.options" :key="keyx" />

    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-6 col-xl-6 col-6">
      <h2>Wybierz schemat kolorów</h2>
     <sketch-picker v-model="colors" v-if="openModal" @input="generateColor"> </sketch-picker>
       {{activeInput}}
       <div v-for="(dataset, i) in chart.data.datasets">
         <h5>Kolor {{i}}</h5>
         <button :style="{background: dataset.backgroundColor[0]}" @click="activeInput = i; openModal = !openModal; hasColor=true" class="button-col"></button>
       </div>
       {{color}}
     
    </div>

  </div>


  <div v-else-if="steps.customizeOwn" class="columns">
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">


      
      <button @click="steps.keysOwn = true; steps.customizeOwn = false" class="btn btn-primary"> Cofnij </button>
      <button @click="steps.customizeOwn = false; steps.finish = true" class="btn btn-primary"
        :disabled="hasColor ? false : true">Dalej </button>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <h2>Spersonalizuj wygląd wykresu</h2>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-6 col-xl-6 col-6">
      <ChartContainer :type="chart.type" :data="chart.data" :id="0" :options="chart.options" :key="keyx" />

    </div>
      <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-6 col-xl-6 col-6">
      <h2>Wybierz schemat kolorów</h2>
      <sketch-picker v-model="colors" v-if="openModal" @input="generateColor"> </sketch-picker>
       {{activeInput}}
       <div v-for="(dataset, i) in chart.data.datasets">
             <h5>Kolor {{i}}</h5>
         <button :style="{background: dataset.backgroundColor[0]}" @click="activeInput = i; openModal = !openModal; hasColor=true" class="button-col"></button>
       </div>
       {{color}}
    </div>

  </div>

  <div v-else class="columns">
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12 navi">
      <button @click="steps.success = false; steps.customize = true" class="btn btn-primary"> Cofnij </button>
      <button @click="generate" class="btn btn-primary">Wygeneruj</button>
    </div>
    <div class="column col-xs-12 col-sm-12 col-md-12 col-lg-12 col-xl-12 col-12">
      <h2>Sukces! Przejdź do swojego wykresu i zobacz efekt!</h2>
    </div>
  </div>
</div>

</template>
<script>
import { Sketch } from 'vue-color'
import cs from '@/api/CrudService'
import ChartContainer from '@/components/ChartContainer'
import router from '@/router/router'
export default {
  name: 'AddChart',
  components: {
    'sketch-picker': Sketch,
    ChartContainer
  },
  data () {
    return {
      colors: '',
      color: '',
      keyx: 10,
      openModal: false,
      activeInput: 0,
      opencolor: false,
      steps: {
        type: false,
        database: true,
        keysOwn: false,
        keysOwnClicked: false,
        customizeOwn: false,
        keys: false,
        keysClicked: false,
        customize: false,
        finish: false
      },
      types: [{
        id: 0,
        human: 'Wykres Liniowy',
        title: 'line',
        image: 'static/line.png'
      },
      {
        id: 1,
        human: 'Wykres Słupkowy',
        title: 'bar',
        image: 'static/bar.png'
      },
      {
        id: 2,
        human: 'Wykres Kołowy',
        title: 'pie',
        image: 'static/pie.png'
      },
      {
        id: 3,
        human: 'Pączek',
        title: 'doughnut',
        image: 'static/doughnut.png'
      }],
      loading: false,
      chart: {
        type: 'line',
        data: {
          labels: [],
          datasets: [{
            label: [],
            data: [],
            backgroundColor: [],
            borderColor: [],
            borderWidth: 1
          }]
        },
        options: {
          scales: {
            xAxes: [{
              ticks: {
                beginAtZero: true
              }
            }]
          }
        }
      },
      chartDataArray: [],
      isValid: false,
      isValidCollection: false,
      isValidDocument: false,
      countryValue: null,
      id: null,
      name: null,
      xhttpData: null,
      age: null,
      gender: null,
      date: null,
      countries: [{name: 'Anglia', value: 'United Kingdom'}, {name: 'Brazylia', value: 'Brazil'}, {name: 'Stany Zjednoczone', value: 'United States'}, {name: 'Portugalia', value: 'Portugal'}, {name: 'Niemcy', value: 'Germany'}, {name: 'Grecja', value: 'Greece'}, {name: 'Włochy', value: 'Italy'}],
      ownChartDataArray: {
        collections: []
      },
      collectionName: '',
      documentName: '',
      documentValue: ''
    }
  },
  methods: {
    deleteCollection (index) {
      this.ownChartDataArray.collections.splice(index, 1)
      if (this.ownChartDataArray.collections.length === 0) {
        this.isValidCollection = false
      }
    },
    deleteDocument (collectionIndex, index) {
      this.ownChartDataArray.collections[collectionIndex].documents.splice(index, 1)
      if (this.ownChartDataArray.collections[collectionIndex].documents.length === 0) {
        this.isValidDocument = false
      }
    },
    collectionAdd () {
      if (this.collectionName.replace(/ /g, '').length === 0) {
        alert('Pole warstwy nie może być puste, podaj co najmniej 3 znaki')
      } else if (this.collectionName.replace(/ /g, '').length < 3) {
        alert('Pole warstwy jest za krótkie, podaj co najmniej 3 znaki')
      } else {
        this.ownChartDataArray.collections.push({collectionName: this.collectionName, documents: []})
        console.log(this.ownChartDataArray)
        this.collectionName = ''
      }
      this.isValidCollection = true
    },
    documentAdd (index) {
      if (this.documentName.replace(/ /g, '').length === 0 || this.documentValue.replace(/ /g, '').length === 0) {
        alert('Pola informacji są nieprawidłowe, wartość i nazwa nie mogą być puste')
      } else {
        this.ownChartDataArray.collections[index].documents.push({documentName: this.documentName, documentValue: this.documentValue})
        this.documentName = ''
        this.documentValue = ''
      }
      this.isValidDocument = true
    },
    useOwnData () {
      var data = []
      var slice
      this.chart.data.labels.length = 0
      this.chart.data.datasets.length = 0
      for (let i = 0; i < this.ownChartDataArray.collections.length; i++) {
        this.ownChartDataArray.collections[i].documents.sort((a, b) => (a.documentName > b.documentName) ? 1 : -1)
        for (let j = 0; j < this.ownChartDataArray.collections[i].documents.length; j++) {
          data.push(this.ownChartDataArray.collections[i].documents[j].documentValue)
          this.chart.data.labels.push(this.ownChartDataArray.collections[0].documents[j].documentName)
        }
        this.chart.data.labels.splice(this.ownChartDataArray.collections[0].documents.length, 1000)
        slice = data.splice(0, this.ownChartDataArray.collections[i].documents.length)
        this.chart.data.datasets.push({
          label: this.ownChartDataArray.collections[i].collectionName,
          data: slice,
          backgroundColor: [],
          borderColor: [],
          borderWidth: 1
        })
        console.log(slice)
        console.log(this.chart.data.labels)
      }
    },
    useData () {
      this.chart.data.labels.length = 0
      this.chart.data.datasets[0].data.length = 0
      for (let i = 0; i < this.chartDataArray.length; i++) {
        this.chart.data.labels.push(`${this.chartDataArray[i].name} (${this.chartDataArray[i].gender})`)
        if (this.chartDataArray[i].gender === 'Mężczyźni') {
          this.chart.data.datasets[0].data.push(this.chartDataArray[i].gotData[0].males)
        } else if (this.chartDataArray[i].gender === 'Kobiety') {
          this.chart.data.datasets[0].data.push(this.chartDataArray[i].gotData[0].females)
        } else if (this.chartDataArray[i].gender === 'Wszyscy') {
          this.chart.data.datasets[0].data.push(this.chartDataArray[i].gotData[0].total)
        }
      }
    },
    resetData () {
      this.chart.data.labels.length = 0
      this.chart.data.datasets[0].data.length = 0
    },
    deleteData (index) {
      if (this.chartDataArray.length === 1) {
        this.isValid = false
      }
      console.log(this.chartDataArray[index])
      this.chartDataArray.splice(this.chartDataArray.indexOf(index), 1)
    },
    checkIsValid () {
      console.log(this.chartDataArray)
      if (this.chartDataArray.length === 0) {
        alert('Nie wypełniłeś wszystkich danych')
      }
    },
    selectCountry (name, value) {
      this.countryValue = value
      this.name = name
    },
    getDataApi () {
      var xhttp = new XMLHttpRequest()
      var _this = this
      xhttp.open('GET', `http://54.72.28.201/1.0/population/${this.date}/${this.countryValue}/${this.age}/?format=json`, false)
      xhttp.onreadystatechange = function () {
        if (xhttp.readyState === 4 && xhttp.status === 200) {
          _this.xhttpData = JSON.parse(xhttp.responseText)
          _this.isValid = true
          var gender = null
          if (_this.gender === 'male') {
            gender = 'Mężczyźni'
          } else if (_this.gender === 'female') {
            gender = 'Kobiety'
          } else if (_this.gender === 'all') {
            gender = 'Wszyscy'
          }
          var chartData = {
            name: _this.name,
            countryValue: _this.countryValue,
            age: parseInt(_this.age),
            gender: gender,
            date: parseInt(_this.date),
            gotData: _this.xhttpData
          }
          _this.chartDataArray.push(chartData)
          console.log(_this.chartDataArray)
          console.log(this.readyState)
        } else if (xhttp.status === 400) {
          alert('Nie wypełniłeś wszystkich danych')
        }
      }
      xhttp.send()
      console.log(this.xhttpData)
    },
    generate () {
      console.log(this.chart)
      cs.addChart(this.chart)
      this.gtl().then((id) => {
        router.push({name: 'chart', params: {id: id, type: this.chart.type}})
      }).catch(e => { console.log('error with promise') })
    },
    generateColor () {
      var color = []
      var colorObj = this.colors.rgba
      Math.decimal = function (n, k) {
        var factor = Math.pow(10, k + 1)
        n = Math.round(Math.round(n * factor) / 10)
        return n / (factor / 10)
      }
      for (var i = 0; i < this.chart.data.datasets[0].data.length; i++) {
        let a = Math.random()
        if (a > 1) {
          a = 1
        } else if (a < 0) {
          a = 0
        } else if (!a || isNaN(a)) {
          a = 1
        }
        if (this.chart.data.datasets.length === 1) {
          color.push('rgba(' + colorObj.r + ', ' + colorObj.g + ', ' + colorObj.b + ', ' + a + ')')
        } else {
          color.push('rgba(' + colorObj.r + ', ' + colorObj.g + ', ' + colorObj.b + ', ' + colorObj.a + ')')
        }
        this.chart.data.datasets[this.activeInput].backgroundColor = color
        this.chart.data.datasets[this.activeInput].borderColor = color
        this.keyx++
      }
    },
    randomData () {
      var arr = []
      for (var i = 0; i < 6; i++) {
        arr.push(Math.random(i))
      }
      return arr
    },
    gtl () {
      return new Promise((resolve, reject) => {
        const request = window.indexedDB.open('charts', 1)
        request.onsuccess = () => {
          const db = request.result
          const transaction = db.transaction(['charts'], 'readonly')
          const invoiceStore = transaction.objectStore('charts')
          const getCursorRequest = invoiceStore.openCursor(null, 'prev')
          getCursorRequest.onsuccess = e => {
            const cursor = e.target.result
            if (cursor) {
              resolve(cursor.value.id)
            }
          }
        }
      })
    },
    setFirst (id) {
      let lea = this.types
      this.types = []
      this.types = lea
      let temp = this.types[0]
      this.types[0] = this.types[id]
      this.types[id] = temp
      this.chart.type = this.types[0].title
    }
    // randomColors () {
    //   var arr = []
    //   for (var i = 0; i < 6; i++) {
    //     var o = Math.round
    //     var r = Math.random
    //     var s = 255
    //     arr.push('rgba(' + o(r() * s) + ',' + o(r() * s) + ',' + o(r() * s) + ',' + r().toFixed(1) + ')')
    //   }
    //   return arr
    // }
  },
  computed: {
    hasColor: function () {
      return this.colors
    }
  }
}
</script>

<style lang="scss">
  .database {
    padding: 0 1rem;
    .content {
      background: pink;
      height: 100%;
    }
  }

.navi {
  justify-content: space-between;
  display: flex;
  .to-right {
    justify-content: end;
  }
}

.menu .menu-item > a > label {
    border-radius: .1rem;
    color: inherit;
    display: block;
    margin: 0 -.4rem;
    padding: .2rem .4rem;
    text-decoration: none;
}

.country-checked{
  background: #f1f1fc;
  color: #5755d9;
  padding: 0.2rem 0.4rem;
}


.type {
  width: 25%;
  float: left;
  border-radius: 2px;
  transition: 1s ease;
  padding: .5rem .3rem;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  background-color: transparent;
  flex-direction: column;
  &:first-of-type {
    width: 50%;
    padding: 15px 25px;
    display: flex;
    justify-content: center;
    
    .image {
      box-shadow: 8px 5px 30px 0 rgba(150, 150, 150, 0.24);
      border-radius: 10px;
      padding: 20px 5px;
    }
  }
  .image {
    max-height: 100%;
    max-width: 100%;
    
    width: auto;
  }
 
}

.shuffleMedium-move {
  transition: width .5s, height .5s, transform .5s;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}
.fade-enter-to, .fade-leave {
  width: 100%;
 height: 300px;
}

.button-col {
  width: 2em;
   height: 2em;
   border: 0;
   border-radius: 100%;
}
</style>