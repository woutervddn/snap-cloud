<template>
    <div class="container-fluid my-5">
      <section id="overview" class="mt-3">
        <div class="row">
          <div class="col-12">
            <h3 class="fw-light">Overview <a class="btn btn-primary text-white float-end" @click="showNewDeviceModal=true"><i class="bi bi-broadcast-pin ml-2 mr-2"></i> Add Device</a></h3>
            <transition-group tag="div" name="fade-slide" appear class="row">
              <div class="col-lg-3" v-for="(card, index) in overviewCards" :key="card.title" :style="{'transition-delay': .2 * index + 's'}">
                  <OverviewCard :title="card.title" :icon="card.icon" :amount="card.amount" :amountsuffix="card.amountsuffix" :color="card.color" :optionalStyles="card.optionalStyles" />
              </div>
            </transition-group>
          </div>
        </div>
        <div class="row">
          <div class="col-lg-8">
            <div class="card text-bg-light text-light bg-opacity-25 mb-3">
              <div class="card-body">    
                <div class="map" style="height:70vh; width:100%; border-radius:5px; overflow: hidden">
                  <l-map ref="map" v-model:zoom="zoom" :center="[51.161419,5.3547403]" :options="{attributionControl: false}">
                    <l-tile-layer url="https://c.tile.openstreetmap.org/{z}/{x}/{y}.png" layer-type="base" name="OpenStreetMap" className="map-tiles"></l-tile-layer>
                    
                    <template v-for="device of devices.filter(x=>x.location != undefined)">
                      <l-circle-marker :lat-lng="[device.location.lat, device.location.lon]" :name="device.friendlyName" radius="1" fillOpacity="0.8" stroke="false" weight="0" color="#FFFFFF">
                          <l-popup>
                            <strong>{{device.friendlyName}}</strong><br/>
                            <a :href="`https://${device.friendlyName}.skytrace.science/public`" target="liveview">Live Snapshot</a>
                          </l-popup>
                      </l-circle-marker>
                      <l-circle v-if="device.devicetype!='BASE_STATION' && device.active == true && (!tripped(device) && !acknowledged(device))" :lat-lng="[device.location.lat, device.location.lon]" :name="device.friendlyName" radius="600" weight="1" fillColor="#FFFFFF" color="#FFFFFF"></l-circle>
                      <l-circle v-if="device.devicetype!='BASE_STATION' && device.active == true && (tripped(device))" :lat-lng="[device.location.lat, device.location.lon]" :name="device.friendlyName" radius="600" weight="1" fillColor="#dc3545" color="#dc3545"></l-circle>
                      <l-circle v-if="device.devicetype!='BASE_STATION' && device.active == true && (!tripped(device) && acknowledged(device))" :lat-lng="[device.location.lat, device.location.lon]" :name="device.friendlyName" radius="600" weight="1" fillColor="#17a2b8" color="#17a2b8"></l-circle>
                      <l-circle v-if="device.devicetype=='BASE_STATION'" :lat-lng="[device.location.lat, device.location.lon]" :name="device.friendlyName" radius="1000" weight="1" fillColor="#17a2b8" color="#17a2b8"></l-circle>

                      <l-circle v-if="device.active == false" :lat-lng="[device.location.lat, device.location.lon]" :name="device.friendlyName" radius="150" weight="1" color="#f5b218" fillColor="#28a745" opacity="0.6" fillOpacity="0.2"></l-circle>
                    </template>
  
                  </l-map>
                </div>
              </div>
            </div>
          </div>
          <div class="col-lg-4">
            <transition name="fade-slide" style="transition-delay: .2s" appear>
              <RecentMatches v-if="detections.length && devices.length" :detections="detections.slice(0,10)" :devices="devices" @reload-detections="fetchData"/>
            </transition>
          </div>
          
        </div>
      </section>
      <section class="mt-5">
        <h3>DEVICES</h3>
        <div class="row">
          <div class="col-lg-6">
            <table class="table table-sm no-table-bg text-white" style="user-select:none;">
                <thead>
                <tr>
                    <th class="text-light" scope="col">id</th>
                    <th class="text-light" scope="col">name</th>
                    <th class="text-light text-right" scope="col">active</th>
                    <th class="text-light text-right" scope="col">deploy</th>
                </tr>
                </thead>
                <tbody>
                    <template v-for="(device,key) in devices">
                       <tr>
                        <th class="text-white">{{ device.id }}</th>
                        <td class="text-white">{{ device.friendlyName }}</td>
                        <td class="text-white text-right">{{ device.active || 'FALSE' }}</td>
                        <td class="text-right fs-3">
                          <a v-if="device.active != true" class="text-primary" @click="deploy(device.id)"><i class="bi bi-power"></i></a>
                          <a v-if="device.active == true" class="text-danger" @click="remove(device.id)"><i class="bi bi-x-octagon-fill"></i></a>
                        </td>
                       </tr>
                    </template>
                </tbody>
            </table>
          </div>
        </div>
      </section>
  </div>
  <div class="modal fade text-dark" :class="{'show':showNewDeviceModal}" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content" style="border-radius: 1rem;">
        <div class="modal-header">
          <h5 class="modal-title">Add new Device</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" @click="showNewDeviceModal=false"></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label for="devicename" class="form-label">Device Name</label>
            <input v-model="newDevice.friendlyName" type="text" class="form-control" id="devicename" placeholder="SNAP02">
          </div>
          <div class="mb-3">
            <label for="loraID" class="form-label">LoRa id</label>
            <input v-model="newDevice.loraID" type="text" class="form-control" id="loraID" placeholder="x001">
          </div>
          <div class="mb-3">
            <label for="satID" class="form-label">Sat id</label>
            <input v-model="newDevice.satID" type="text" class="form-control" id="satID" placeholder="x001">
          </div>
          <div class="mb-3">
            <label for="devicetype" class="form-label">Devicetype</label>
            <select class="form-select" id="devicetype" v-model="newDevice.devicetype">
              <option value="snapdevice" selected>Snap Device</option>
              <option value="zigbee">Zigbee</option>
            </select>
          </div>
          <div class="mb-3">
            <label for="availableTriggers" class="form-label">Available Triggers</label>
            <select multiple class="form-select" id="availableTriggers" v-model="newDevice.availableTriggers">
              <option value="PIR" selected>pir</option>
              <option value="Seismo">Seismo</option>
              <option value="Piezo">Piezo</option>
            </select>
          </div>
          <div class="form-check">
            <input class="form-check-input" type="checkbox" value="" id="hasCam">
            <label class="form-check-label" for="hasCam">
              Camera available
            </label>
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" @click="showNewDeviceModal=false">Close</button>
          <button type="button" class="btn btn-primary" @click="addDevice">Add Device</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import "leaflet/dist/leaflet.css";
import L from 'leaflet';
import { LMap, LTileLayer, LPolygon, LCircle, LCircleMarker, LPopup, LMarker, LIcon, LPolyline, LTooltip } from "@vue-leaflet/vue-leaflet";

import OverviewCard from '@/components/OverviewCard.vue';
import RecentMatches from '@/components/RecentMatches.vue';
import Impact from '@/components/Impact.vue';

export default {
  name: 'Dashboardview',
  components: {
    OverviewCard,
    RecentMatches,
    Impact,
    LMap,
    LTileLayer,
    LPolygon,
    LCircle,
    LCircleMarker,
    LPopup,
    LMarker,
    LIcon,
    LPolyline,
    LTooltip
  },
  methods:{
    fetchData(){

      // FETCH DEVICES
      fetch(`https://api.vierpuntnul.be/devices`
      ).then(
        res=>res.json()
      ).then((response) => {
        console.log("devices", response);
        this.devices = response;
        this.overviewCards[ this.overviewCards.findIndex(x=>x.title.indexOf('Active Sensors')>-1) ].amountsuffix = `/${this.devices.length}`;
        this.overviewCards[ this.overviewCards.findIndex(x=>x.title.indexOf('Active Sensors')>-1) ].amount = this.devices.filter(x=>x.active).length;
      }).catch( (error) => {
        console.log(error);
      });

      // FETCH DETECTIONS
      fetch(`https://api.vierpuntnul.be/detections`
      ).then(
        res=>res.json()
      ).then((response) => {
        console.log("detections", response);
        this.detections = response.reverse();
        this.overviewCards[ this.overviewCards.findIndex(x=>x.title.indexOf('Total Detection')>-1) ].amount = this.detections.length;
        this.overviewCards[ this.overviewCards.findIndex(x=>x.title.indexOf('Detections Today')>-1) ].amount = 
          this.detections.map(x=>x.timestamp).filter(obj => new Date(obj).setHours(0,0,0,0) == new Date().setHours(0,0,0,0)  ).length        
      }).catch( (error) => {
        console.log(error);
      });
    },
    tripped(device){
      console.log('tripped', this.detections.filter(x=> (x.state === "TRIPPED" && x.deviceId == device.id )).length);
      return this.detections.filter(x=> (x.state === "TRIPPED" && x.deviceId == device.id )).length
    }
    ,
    acknowledged(device){
      console.log('acknowledged', this.detections.filter(x=> (x.state === "ACKNOWLEDGED" && x.deviceId == device.id )).length);
      return this.detections.filter(x=> (x.state === "ACKNOWLEDGED" && x.deviceId == device.id )).length
    },
    addDevice(){
      fetch(`https://api.vierpuntnul.be/devices/`,{
                headers: {
                    'Accept': 'application/json',
                    'Content-Type': 'application/json'
                },
                method: 'POST',     
                body: JSON.stringify(this.newDevice)
            }
        ).then(
            res=>res.json()
        ).then((response) => {
            console.log("addDevice:", response);
            this.showNewDeviceModal=false;
            this.fetchData();

        }).catch( (error) => {
            console.log(error);
        });
    },
    deploy(deviceId){
      let myapp= this;
      navigator.geolocation.getCurrentPosition(function(data){
        fetch(`https://api.vierpuntnul.be/devices/${deviceId}`,{
                headers: {
                    'Accept': 'application/json',
                    'Content-Type': 'application/json'
                },
                method: 'PATCH',     
                // body: JSON.stringify({
                //   'active':true,
                //   'location':{
                //     'lat': data.coords.latitude,
                //     'lon': data.coords.longitude
                //   }
                body: JSON.stringify({
                  'active':true
                })
            }
        ).then(
            res=>res.json()
        ).then((response) => {
            console.log("deploy:", response);
            console.log(myapp.fetchData);
            myapp.fetchData();
        }).catch( (error) => {
            console.log(error);
        })
      },function(err){
        if(err){
          alert('PLEASE ACCEPT GEOLOCATION');
        }
      } );
    },
    remove(deviceId){
      fetch(`https://api.vierpuntnul.be/devices/${deviceId}`,{
              headers: {
                  'Accept': 'application/json',
                  'Content-Type': 'application/json'
              },
              method: 'PATCH',     
              body: JSON.stringify({
                'active':false
              })
          }
      ).then(
          res=>res.json()
      ).then((response) => {
          console.log("remove:", response);
          this.fetchData();
      }).catch( (error) => {
          console.log(error);
      })
    }
  },
  mounted(){
    this.fetchData();
    this.timer = setInterval(this.fetchData, 10 * 1000);
  },
  beforeUnmount () {
    this.cancelAutoUpdate();
  },
  data(){
    return {
      newDevice:{
        devicetype: 'snapdevice'
      },
      timer: '',
      devices:[],
      detections: [],
      overviewCards: [
        { 
          title: 'Detections Today',
          amount: 20,
          amountsuffix: '',
          icon: 'bi-bell-fill',
          color: 'bg-primary'
        },
        { 
          title: 'Total Detections',
          amount: 247,
          amountsuffix: '',
          icon: 'bi-bell-fill',
          color: 'bg-success'
        },
        { 
          title: 'Active Sensors',
          amount: 4,
          amountsuffix: '/4',
          icon: 'bi-broadcast',
          color: 'bg-warning'
        }
      ],
      zoom: 14,
      liveTimer: '',
      showNewDeviceModal: false
    }
  }
}
</script>

<style>
  .fade-slide-enter-from{
    opacity: 0;
    transform: translateY(-60px)
  }
  .fade-slide-enter-to{
    opacity: 1;
    transform: translateY(-0px)
  }
  .fade-slide-enter-active{
    transition: all 0.4s ease;
  }

  :root {
    /* --map-tiles-filter: brightness(0.6) invert(1) contrast(3) hue-rotate(200deg) saturate(0.3) brightness(0.7); */
    --map-tiles-filter: brightness(0.7) invert(1) contrast(3) hue-rotate(180deg) saturate(0.2) brightness(0.3);
  }
  .map-tiles {
    filter:var(--map-tiles-filter, none);
  }
  .planeIcon{
    filter: invert(1);
    opacity:0.6;
  }
  .planeIconMatched{
    filter: invert(0.82) sepia(72%) saturate(7415%) hue-rotate(358deg) brightness(100%) contrast(122%);;
  }
  .fade-slide-enter-active{
    transition: all 0.4s ease;
  }
  .show{
    display:block !important;
  }
</style>