
<template>
  <div>
     <div class="invoice" ref="invoiceContent">
        <header>
            <h1>Facture</h1>
        </header>
        <div class="bill-to">
            <p><strong>Facturé à :</strong></p>
            <p>Nom du client</p>
            <p>Adresse du client</p>
            <p>Ville, Code postal</p>
        </div>
        <table>
            <thead>
                <tr>
                    <th>Description</th>
                    <th>Quantité</th>
                    <th>Prix unitaire</th>
                    <th>Total</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Article 1</td>
                    <td>2</td>
                    <td>$10.00</td>
                    <td>$20.00</td>
                </tr>
                <tr>
                    <td>Article 2</td>
                    <td>1</td>
                    <td>$15.00</td>
                    <td>$15.00</td>
                </tr>
            </tbody>
        </table>
        <div class="total">
            <p><strong>Total à payer : $35.00</strong></p>
        </div>
    </div>
    <button @click="genererPDF">Générer un PDF</button>
    
    <hr>


      <div v-for="weather in weatherData" :key="weather.id">
        <h1 class="name">{{ weather.name }}</h1>
      </div>
    <div class="weather" > 
      <div v-for="weather in weatherData.list" :key="weather.id" class="data">
        <img :src="getWeatherIcon(weather)" alt="">
        <div class="donner">
          <h1>{{ weather.weather[0].main }}</h1>
          <h4>{{ formatDateTime(weather.dt_txt) }}</h4>
          <h4>Température: {{ convertor(weather.main.temp) }}°C</h4>
          <h4>Humiditer:  {{ weather.main.humidity}}%</h4>
          <h4>Vitesse de vents:  {{ConvertorMK(weather.wind.speed) }} km/h</h4>
        </div>  
      </div>
    </div>
    <hr>
    <div class="lines" >
      <canvas id="line" height="500" ></canvas>
    </div>
    <div class="Bares" >
      <canvas id="bar" height="500" ></canvas>
    </div>
    <hr>
    <div id="chartdiv" style="width: 100%; height: 300px;"></div>

    <hr>
    <hr>
    <hr>
    <br>

    <div class="termique">
      <div id="chartTherm"></div>
    </div>
      

  </div>
</template>

<script>
import axios from 'axios'
import  html2pdf  from 'html2pdf.js'
import Chart from "chart.js/auto"
import 'chartjs-adapter-date-fns'

import * as am4core from '@amcharts/amcharts4/core'
import * as am4charts from '@amcharts/amcharts4/charts'
import am4theme_animated from '@amcharts/amcharts4/themes/animated'

import ApexCharts from 'apexcharts'





export default {
  name: 'App',
  data(){
    return{
      weatherData: {},
      apiKey: "33e9ba1800bb51d52aef75149c126cf2",
      Charts:{},
      apiKeyMaps: "5iefEgx3GqDq5MxYEzCaIRIcUjDC6U3n",
      Maps: null,
      marker:null,
      center : [2.61590 , 6.48177 ]
      
    }
  },
  mounted(){
    am4core.useTheme(am4theme_animated)
    this.getweather()
    
  },
  methods:{
      
      genererPDF(){
        try{
        const invoiceContent  = this.$refs.invoiceContent 

        const option = {
          margin: 0,
          filename: 'document.pdf',
          image: {type: 'jpeg',quality: 0.98},
          html2canvas: { scale: 2 },
          jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait', },
        }

          html2pdf(invoiceContent,option)

        }catch(error){
          console.log(error)
        }

      },

      async getweather(){
        
          navigator.geolocation.getCurrentPosition(async (position)=>{
            console.log(position)
            const lat = 6.48177  
            const lon = 2.61590 

            const res = await axios.get(`http://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&appid=${this.apiKey}`)
            if (res.status === 200) {
              this.weatherData = res.data
              this.humidityChart()
              this.tempChart()
              this.Am4Chart()
              this.Thermique()
            }
            
          })

          

       
      },
      getWeatherIcon(weather){
        const main = weather.weather[0].main
        const iconCode = weather.weather[0].icon

        const isDayIcon = iconCode.includes("d")

        if (isDayIcon) {
          const weatherIcons =  {
            "Clear" : "./Icons/Animate/day.svg",
            "Clouds" : "./Icons/Animate/cloud.svg",
            "Haze": "./Icons/haze.svg",
            "Rain": "./Icons/Animate/rain.svg",
            "Snow": "./Icons/Animate/snow.svg",
            "Storm": "./Icons/Animate/storm.svg",
          }

          return weatherIcons[main]
        }else{
          const weatherIcons =  {
            "Clear" : "./Icons/Animate/night.svg",
            "Clouds" : "./Icons/Animate/cloud-night.svg",
            "Haze": "./Icons/haze.svg",
            "Rain": "./Icons/Animate/rain.svg",
            "Snow": "./Icons/Animate/snow.svg",
            "Storm": "./Icons/Animate/storm.svg",
          }

          return weatherIcons[main]
        }

        
      },
      convertor(kelvin){
        //Calcule pour convertir la température fr kelvin à Celsius
        return (kelvin - 273.15).toFixed(2) // Arrondi à 2 décimales
      },
      ConvertorMK(ms){
        return (ms * 3.6).toFixed(2)
      },
      formatDateTime(dtTxt) {
      const dateTime = new Date(dtTxt);
      const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', hour: 'numeric', minute: 'numeric', second: 'numeric', timeZoneName: 'short' };
      return dateTime.toLocaleString('fr-FR', options);
      },
      formatDateTimeDay(dtTxt){ //Uniquement le jour
        const dateTime = new Date(dtTxt);
        const options = { weekday: 'long' }; // Seule l'option 'weekday' est spécifiée
        return dateTime.toLocaleString('fr-FR', options);
      },
      formatDateTimeHours(dtTxt){ //Uniquement l'heure
        const dateTime = new Date(dtTxt);
        const options = { hour: 'numeric', minute: 'numeric' }; // Seules les options 'hour' et 'minute' sont spécifiées
        return dateTime.toLocaleString('fr-FR', options);
      },
      humidityChart(){

        const humidityData = this.weatherData.list.map(weather => weather.main.humidity) //tableau humidité
        const dt = this.weatherData.list.map(weathers => weathers.dt_txt) // tableau date
        const ctx = document.querySelector('#line') // graphe
        new Chart(ctx,{
          type: 'line',
          data:{
            labels: dt,
            datasets:[
              {
                label:'Humidité',
                data: humidityData,
                borderColor:'rgba(75, 192, 192, 1)',
                borderWidth: 3,
                fill: false
              }
            ]
          },
          options:{
            responsive: true,
            maintainAspectRatio: false,
            scales:{
              x: {
                type:'time',
                time:{
                  parser: 'yyyy-MM-dd HH:mm:ss',
                  unit:'hour',
                  displayFormats:{
                    hour:'yyyy-MM-dd HH:mm'
                  },
                },
                title:{
                  display: true,
                  text: 'Date et heure'
                },
              },
              y:{
                title:{
                  display: true,
                  text:'Himidité (%)'
                }
              }
            }
          }

        }) 
      },
      tempChart(){
        const Temp = this.weatherData.list.map(weather=> this.convertor(weather.main.temp))
        const dt = this.weatherData.list.map(weathers => weathers.dt_txt)
        const ctx = document.querySelector('#bar')
        new Chart(ctx,{
          type:'bar',
          data:{
            labels: dt,
            datasets:[
              {
                label:'Température',
                data: Temp,
                borderWidth: 3,
                fill:true
              }
            ]
          },
          options:{
            responsive: true,
            maintainAspectRatio: false,
            scales:{
              x: {
                type:'time',
                time:{
                  parser: 'yyyy-MM-dd HH:mm:ss',
                  unit:'hour',
                  displayFormats:{
                    hour:'yyyy-MM-dd HH:mm'
                  },
                },
                title:{
                  display: true,
                  text: 'Date et heure'
                },
              },
              y:{
                title:{
                  display: true,
                  text:'Himidité (%)'
                }
              }
            }
          }
        })
      },
      Am4Chart(){
        
        let chart = am4core.create("chartdiv",am4charts.XYChart)
        const humidityData = this.weatherData.list.map(weather => weather.main.humidity)
        const date = this.weatherData.list.map(weather_dt => weather_dt.dt_txt)
        let chartData = []

        for(let i = 0; i < humidityData.length; i++){
          chartData.push({Humidite:humidityData[i], Dates: date[i]})
        }

        chart.data = chartData

        let HumidityAxis = chart.yAxes.push(new am4charts.ValueAxis())
        HumidityAxis.title.text = "Humidité (%)"

        let DateAxis = chart.xAxes.push(new am4charts.DateAxis())
        DateAxis.title.text = "Date"
        DateAxis.renderer.minGridDistance = 50;

        let series = chart.series.push(new am4charts.ColumnSeries())
        series.dataFields.dateX = "Dates"
        series.dataFields.valueY  = "Humidite"

        chart.cursor = new am4charts.XYCursor()

      },

      Thermique(){
        const Temp = this.weatherData.list.map(weather => this.convertor(weather.main.temp))
        const Day = this.weatherData.list.map(weather => this.formatDateTimeDay(weather.dt_txt))
        const Hours = this.weatherData.list.map(weather => this.formatDateTimeHours(weather.dt_txt))
        console.log(Temp)
        console.log(Hours)
        console.log(Day)

        const seriesData = []

        const dayDataMap = {}

        Day.forEach((day,index)=>{
          const hour = Hours[index]
          const temp = Temp[index]

          if (!dayDataMap[day]) {
            dayDataMap[day] = []
          }

          dayDataMap[day].push({
            x: hour,
            y: temp
          })

        })

        for(const day in dayDataMap){
          seriesData.push({
            name: day,
            data: dayDataMap[day]
          })
        }

        // Day.forEach((day)=>{
        //   const dayData = []
        //   Hours.forEach((hour,i)=>{
        //     if (Day[i] === day) {
        //       dayData.push({
        //         x: hour,
        //         y: Temp[i]
        //       })
        //     }
        //   })
        //   seriesData.push({
        //     name: day,
        //     data: dayData
        //   })
        // })
        
        const options = {
          
          chart:{
            height:600,
            width:'100%',
            type:'heatmap'
          },
          plotOptions: {
            heatmap: {
              shadeIntensity: 0.5,
              radius: 0,
              useFillColorAsStroke: false,
              colorScale: {
                ranges: [
                  {
                    from: -30,
                    to: 5,
                    name: 'basse',
                    color: '#00A100'
                  },
                  {
                    from: 6,
                    to: 20,
                    name: 'modérée',
                    color: '#128FD9'
                  },
                  {
                    from: 21,
                    to: 45,
                    name: 'élevée',
                    color: '#FFB200'
                  },
                  {
                    from: 46,
                    to: Infinity,
                    name: 'très élevée',
                    color: '#FF0000'
                  }
                ]
              }
            }
          },
          series: seriesData,
          dataLabels: {
            enabled: false
          },
          stroke: {
            width: 1
          },
          title: {
            text: 'HeatMap Chart with Color Range Température'
          },
        }
        const Chart = new ApexCharts(document.querySelector('#chartTherm'),options)
        Chart.render()
        
      }
      
      
      
      

  },
 
    
    
    
  
}
</script>

<style scoped>

 body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    width: 100%;
}
.invoice {
  max-width: 100%;
  width: auto;
  margin: 0 auto;
  padding: 20px;
  background: #f9f9f9;
}
.invoice header {
    padding: 10px 0;
    text-align: center;
    background: #f0f0f0;
}
.invoice .bill-to {
    margin-top: 20px;
}
.invoice table {
    width: 100%;
    border-collapse: collapse;
}
.invoice table th, .invoice table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}
.invoice .total {
    margin-top: 20px;
    text-align: right;
}

.weather{
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 30px;
}
.weather .data{
  background: #fff;
  padding: 20px;
  display: flex;
  align-items: center;
  justify-content: space-around;
  gap: 20px;
  width: 500px;
  box-shadow: 0 0 10px rgba(0, 0, 0, .5);
  border-radius: 20px;
}
img{
  width: 200px;
}
.name{
  font-size: 48px;
}
.lines{
  margin-top: 80px;
}
.Bares{
  margin-top: 80px;
}
       
</style>
