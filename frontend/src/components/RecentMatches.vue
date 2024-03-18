<template>
    <div class="card text-bg-light text-light mb-3 bg-opacity-25">
        <!-- <div class="card-header">Header</div> -->
        <div class="card-body">
            <h2 class="card-title mb-5">Recent Detections
            <!-- <a class="btn btn-outline-light float-end" @click="toggleView">{{currentStateString}}</a> -->
            </h2>
            <table class="table table-sm no-table-bg text-white" style="user-select:none;">
                <thead>
                <tr>
                    <th class="text-light" scope="col">Timestamp</th>
                    <!-- <th class="text-light" scope="col">Device ID</th> -->
                    <th class="text-light" scope="col">Device Name</th>
                    <th class="text-light text-right" scope="col">State</th>
                    <th class="text-light text-right" scope="col"></th>
                </tr>
                </thead>
                <tbody>
                    <template v-for="(detection,key) in detections.filter(det=> devices.find(dev=>dev.id === det.deviceId).active)">
                        <tr :class="{'bg-muted':detection.state=='RESOLVED','bg-info':detection.state=='ACKNOWLEDGED','bg-danger':detection.state=='TRIPPED', 'active':activeDetection===detection.id}" @click="toggleDetection(detection)">
                            <th class="text-white" scope="row">{{ new Date(detection.timestamp).toLocaleString('nl-BE') }}</th><!-- This transforms it to GMT -->
                            <!-- <td class="text-white">{{ detection.deviceId }}</td> -->
                            <td class="text-white text-center">{{ devices?.find(x => x.id === detection.deviceId)?.friendlyName }}</td>
                            <td class="text-white text-right">{{ detection.state }}</td>
                            <td class="text-white text-right" v-if="activeDetection===detection.id"><i class="bi bi-arrow-down-circle-fill"></i></td>
                            <td class="text-white text-right" v-if="activeDetection!==detection.id"><i class="bi bi-arrow-left-short"></i></td>
                            <!-- <td>
                                <MovingImagePopup :matches="value"/>
                            </td> -->
                        </tr>
                        <tr class="datarow" :class="{'visible':activeDetection===detection.id}">
                            <td colspan="4">
                                <div :class="{'visible':activeDetection===detection.id}" class="fw-bold text-white">
                                Detection No. {{detection.id }} <br/>
                                Device info: {{ devices?.find(x=>x.id == detection.deviceId)?.id }} - {{ devices.find(x=>x.id == detection.deviceId)?.friendlyName }} <br/>
                                Device type: {{ devices?.find(x=>x.id == detection.deviceId)?.type }} <br/>
                                Triggered by: {{ detection.triggerInfo.trigger }} <br/>
                                <!-- <img v-if="devices?.find(x=>x.id == detection.deviceId)?.hasCam == true" :src="detection.triggerInfo.imageUrl" class="mt-3"/> <br/><br/> -->
                                <iframe v-if="devices?.find(x=>x.id == detection.deviceId)?.hasCam == true" src="http://192.168.1.153:81/stream" class="mt-3 stream"></iframe> <br/><br/>
                                <div class="btn-group btn-block mb-4" role="group">
                                    <button type="button" class="btn btn-primary" @click="ack(detection.id)">ACK</button>
                                    <button type="button" class="btn btn-secondary" @click="reso(detection.id)">RESO</button>
                                </div>
                            </div>
                            </td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
// import ImagePopup from '@/components/ImagePopup.vue';
// import MovingImagePopup from '@/components/MovingImagePopup.vue'

export default {
    components:{
        // ImagePopup,
        // MovingImagePopup
    },
    data(){
        return {
            activeDetection: ''
        }
    },
    props:{
        detections:{},
        devices:{}
    },
    methods:{
        toggleDetection(detection){
            if(this.activeDetection != detection.id ){
                this.activeDetection=detection.id
            } else {
                this.activeDetection = '';
            }
        },
        ack(detectionId){
            fetch(`https://api.vierpuntnul.be/detections/${detectionId}`,{
                    headers: {
                        'Accept': 'application/json',
                        'Content-Type': 'application/json'
                    },
                    method: 'PATCH',                                                              
                    body: JSON.stringify({'state':'ACKNOWLEDGED'})
                }
            ).then(
                res=>res.json()
            ).then((response) => {
                console.log("resolvement:", response);
                this.$emit('reload-detections');
            }).catch( (error) => {
                console.log(error);
            });

        },
        reso(detectionId){
            fetch(`https://api.vierpuntnul.be/detections/${detectionId}`,{
                    headers: {
                        'Accept': 'application/json',
                        'Content-Type': 'application/json'
                    },
                    method: 'PATCH',                                                              
                    body: JSON.stringify({'state':'RESOLVED'})
                }
            ).then(
                res=>res.json()
            ).then((response) => {
                console.log("resolvement:", response);
                this.$emit('reload-detections');
            }).catch( (error) => {
                console.log(error);
            });
        }
    }
}
</script>

<style>
.datarow{
    height:0;
    opacity:0;
    display: none;
    transition: all 0.4s ease;
}
.datarow.visible{
    height:auto;
    opacity: 1;
    display: table-row;
    transition: all 0.4s ease;
}
.active tr,
.active td,
.active th{
    color: #ffc107!important
}
img, iframe{
    max-width: 100%;
}
.stream{
    height:240px;
    width:320px;
    display:block;
    margin:0 auto;
    
}
</style>