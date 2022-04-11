<template>
  <div>

    <div>
      <WorldMap @map-clicked="what" />
    </div>
    <input type="text" :value="geoPosition">
    <button @click="searchCity">Add</button>
    <div id="clocks">
      <Clock   v-for="(c,pos) in selectedCities" :key="pos"
        :time-zone="c.timeZone" :label="c.name" :position="pos" @click.native="deleteClock(pos)">
      </Clock>
    </div>
    <button @click="saveLocations"> save locations </button>
  </div>
</template>

<script lang="ts">
// import ChildComponent from "./60-props-child.vue";
import axios, { AxiosResponse } from "axios";
import Clock from "./70-clock.vue";
import WorldMap from "./70-world-map.vue";
import { Vue, Component } from "vue-property-decorator";
import { getAuth } from "@firebase/auth";
import {CollectionReference, updateDoc, query, where, setDoc,doc, addDoc, getDocs, DocumentSnapshot, deleteDoc} from "@firebase/firestore";
import { initializeApp } from "@firebase/app";
import { getFirestore,collection} from "firebase/firestore";


const timezoneDBUrl = "http://api.timezonedb.com/v2.1";



const firebaseConfig = {
  apiKey: "AIzaSyDKXUdRUlbOiY3SCHzCenHbOaEYvWIG0ts",
  authDomain: "project4-76056.firebaseapp.com",
  projectId: "project4-76056",
  storageBucket: "project4-76056.appspot.com",
  messagingSenderId: "717694181905",
  appId: "1:717694181905:web:c9add60493037c87f407b0",
  measurementId: "G-RW29XM0P0J"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

type City = {
  name: string;
  timeZone: string;
};

type TimeZoneData = {
  countryName: string;
  gmtOffset: number;
  regionName: string;
  zoneName: string;
};
@Component({ components: { Clock, WorldMap } })
export default class Sample extends Vue {
  geoPos: { lat?: number; lng?: number } = {};
  selectedCities: Array<City> = [];
  apiKey = "";
  mounted(): void {
    this.apiKey = process.env.VUE_APP_TIMEZONE_API_KEY;
    

    //here
    //let c :City = {name: "Ohio", timeZone:"America/New_York"};
    //this.selectedCities.push(c);

    const auth = getAuth();
    let uid = auth.currentUser?.uid;
    const docRef = collection(db, "Users",uid +"", "locations");
    const querey =  getDocs(docRef).then((querey) => {

      querey.docs.forEach((ds :DocumentSnapshot) =>{
        const data = ds.data();
        
        let city : City= {
          name: data?.name,
          timeZone: data?.timeZone
        }
        this.selectedCities.push(city);
        console.log(city);
        console.log(this.selectedCities);
        console.log("selected cities");
      } );
     
      });

  }

  deleteClock(pos: number) : void{

    try{
    const auth = getAuth();
    let uid = auth.currentUser?.uid;
    const colRef = collection(db, "Users",uid +"", "locations");
    const querey =  getDocs(colRef).then((querey) => {

      querey.docs.forEach((ds :DocumentSnapshot) =>{
        const data = ds.data();
        let city :  City= {
          name: data?.name,
          timeZone: data?.timeZone
        }
        console.log(city);
        console.log("DB");
        console.log(this.selectedCities[pos]);
        console.log("Selected cities");
       
          let ref = ds.ref;
          deleteDoc(ref);
          console.log("deleted");
          

       

      } );
     
      });

    

   }catch{
     console.error;

    }
    this.selectedCities.splice(pos,1);
    this.saveLocations();


  }

  

  get geoPosition(): string {
    if (this.geoPos.lat && this.geoPos.lng)
      return `${this.geoPos.lat.toPrecision(5)},${this.geoPos.lng.toPrecision(
        5
      )}`;
    else return "N/A";
  }
  searchCity(): void {
    const param = new URLSearchParams();
    param.append("key", this.apiKey);
    param.append("format", "json");
    param.append("by", "position");
    param.append("lat", this.geoPos.lat!.toString());
    param.append("lng", this.geoPos.lng!.toString());
    const tzUrl = `${timezoneDBUrl}/get-time-zone?` + param.toString();
    // Use a Web Proxy Server to get around CORS issue
    // since timezonedb.com does not allow CORS
    axios
      .request({
        method: "GET",
        url: "https://api.allorigins.win/get",
        params: {
          url: tzUrl,
        },
      })
      .then((r: AxiosResponse) => {
        return r.data;
      })
      .then((r: any) => JSON.parse(r.contents))
      .then((r: TimeZoneData) => {
        // Add the selected location to our array

        let notfound = true;
        for(let i in this.selectedCities){
          if(this.selectedCities[i].name== r.regionName && this.selectedCities[i].timeZone == r.zoneName){
            notfound = false;
            break;
          }
        } 
        if(notfound){
        this.selectedCities.push({ name: r.regionName, timeZone: r.zoneName });
        }
      });
  }

  what(geoPos: { lat: number; lng: number }): void {
    // When the user pans the map left/right the longitude
    // angle can be out of the [-180,+180] range
    while (geoPos.lng > 180) geoPos.lng -= 360;
    while (geoPos.lng < -180) geoPos.lng += 360;
    this.geoPos = { ...geoPos };
  }

  saveLocations() :void{
   
    const auth = getAuth();
    let uid = auth.currentUser?.uid;

    //const userCol = collection(db, "Users" + ""+ uid);
    const userCol = collection(db, "Users");
    
    let i =0;
    const docRef = collection(db, "Users",uid +"", "locations");

    for(let i in this.selectedCities){
      addDoc(docRef,this.selectedCities[i]);
      console.log(this.selectedCities[i]);
      
    }
    
    console.log(uid)

    
  

  }
}
</script>

<style scoped>
#clocks {
  margin-top: 1em;
}
pre {
  white-space: normal;
  padding: 0.5em;
  border: 2px solid gray;
}
</style>