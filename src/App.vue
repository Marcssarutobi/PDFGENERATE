
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
      <div class="lines" >
        <canvas id="line" height="500" ></canvas>
      </div>
      <div class="Bares" >
        <canvas id="bar" height="500" ></canvas>
      </div>


      

  </div>
</template>

<script>
import axios from 'axios'
import  html2pdf  from 'html2pdf.js'
import Chart from "chart.js/auto"
import 'chartjs-adapter-date-fns'





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
