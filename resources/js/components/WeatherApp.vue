<template>
    <div class="text-white mb-8">
        <div class="place-input text-gray-800">
           <input type="search" id="address" class="form-control" placeholder="Where are we going?" />

            <p>Selected: <strong id="address-value">none</strong></p>
        </div>

        <div class="weather-container font-sans w-128 max-w-lg overflow-hidden bg-gray-900 shadow-lg mt-4">
                <div class="current-weather flex items-center justify-between px-6 py-8">
                       <div class="flex items-center">
                           <div>
                           <div class="text-6xl font-semibold">
                               {{currentTemperature.actual}}°C
                           </div>
                           <div>Fell like {{ currentTemperature.feels }}°C</div>
                           </div>
                           <div class="mx-5">
                               <div class="font-semiblod">
                                   {{ currentTemperature.summary }}
                               </div>
                               <div>{{ location.name }}</div>
                           </div>
                       </div>
                       <div>
                           <canvas id="iconC" width="96" height="96"></canvas>
                       </div>
                </div>
                <div class="future-weather text-sm bg-gray-800 px-6 py-9 overflow-hidden">
                    <div v-for="(day,index) in daily" :key='day.time' :class="{'mt-8':index>0}"  
                        v-if="index < 5"
                        class="flex items-center">
                        <div class="w-1/6 text-lg text-gray-200">{{  toDay(day.time) }}</div>
                        <div class="w-4/6 px-4 flex items-center">
                            <div>

                                 <canvas :id="`icon${index+1}`"  :data-icon="toKebabCase(day.icon)" width="30" height="30"></canvas>
                            </div>
                            <div class="ml-3">{{day.summary}}</div>
                          
                        </div>
                        <div class="w-1/6 text-right">
                            <div>
                               {{ Math.round(day.temperatureHigh)}} °C
                            </div>
                            <div>
                          {{ Math.round(day.temperatureLow)}} °C

                            </div>
                        </div>
                    </div>
          
                </div>
        </div>
    </div>
</template>

<script>
    export default {
        mounted() {
             this.fetchData();
            var placesAutocomplete = places({
    appId: 'plLWNHNL4TB4',
    apiKey: '41beba81535bbacaa1220216aad156ac',
    container: document.querySelector('#address')
  }).configure({
      type:'city',
      aroundLatLngViapIP:false
  });

  var $address = document.querySelector('#address-value')
  placesAutocomplete.on('change', (e)=> {
    $address.textContent = e.suggestion.value;
     this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`;
    this.location.lat =e.suggestion.latlng.lat;
    this.location.lng =e.suggestion.latlng.lng;
  });

  placesAutocomplete.on('clear', function() {
    $address.textContent = 'none';
   
  });
         
        },
        watch:{
            location:{
                handler(newValue,oldValue)
                {
                    this.fetchData();
                },
                deep:true
            }  
        },
        data(){
            return{
                currentTemperature:{
                    actual :'',
                    feels:'',
                    summary:'',
                    icon:'',
                },
                location:{
                    name :'Toronto, Canada',
                    lat:43.6532,
                    lng:-79.38323,
                },
                daily:[],
            }
        },
        methods:{
            fetchData()
            {
                var skycons = new Skycons({'color': 'pink'});
                fetch(`/api/weather?lat=${this.location.lat}&lng=${this.location.lng}`).then(respone=>respone.json()).then(data=>{
                    // console.log(data);

                    this.currentTemperature.actual = Math.round(data.currently.temperature);
                    this.currentTemperature.feels = Math.round(data.currently.apparentTemperature);
                    this.currentTemperature.summary = data.currently.summary;

                    this.currentTemperature.icon = this.toKebabCase(data.currently.icon);
                
                                  this.daily = data.daily.data

                skycons.add("iconC", this.currentTemperature.icon);
                  skycons.play();

                    this.$nextTick(()=>{
                   
                 skycons.add("icon1",   document.getElementById('icon1').getAttribute('data-icon'));
                 skycons.add("icon2",   document.getElementById('icon2').getAttribute('data-icon'));
                 skycons.add("icon3",   document.getElementById('icon3').getAttribute('data-icon'));
                 skycons.add("icon4",   document.getElementById('icon4').getAttribute('data-icon'));
                 skycons.add("icon5",   document.getElementById('icon5').getAttribute('data-icon'));

                 skycons.play()
                  
                    })
                  

                })
            },
            toKebabCase(stirng){
                return stirng.split(' ').join('-');
            },
            toDay(time){
                const newdate = new Date(time*1000);
                const days=['SUN','MON' ,'TUE','WED','THU','FRI','SAT'];


                return days [newdate.getDay()];
                }
            
        }
    }
</script>
