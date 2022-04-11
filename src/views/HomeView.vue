<template>
  <div id="app">
    <h2>This is the main page. I got here via {{byWayOf}}</h2>
    <p>{{userInfo}}</p>
    <img :src="userPhotoURL" v-if="userPhotoURL.length > 0" width="64">
    <button @click="outtahere">Logout</button>
    <button @click="deleteBttn">Delete Account</button>
    <worldTime>

    </worldTime>
    

  </div>
</template>

<script lang="ts">


import { Component, Prop, Vue } from "vue-property-decorator";
import worldTime from "./70-world-time.vue";
import {
  getAuth,
  onAuthStateChanged,
  User,
  Auth,
  signOut,
  deleteUser,
} from "firebase/auth";
import { initializeApp } from "firebase/app";
import { collection, deleteDoc, doc, DocumentSnapshot, getDocs, getFirestore } from "firebase/firestore";



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

@Component({
  components:{
  worldTime
  },

})
export default class HomeView extends Vue {
  @Prop() readonly byWayOf!: string;

  userPhotoURL = "";
  auth: Auth | null = null;
  userInfo = "";
  mounted(): void {
    this.auth = getAuth();
    onAuthStateChanged(this.auth, (user: User | null) => {
      //console.log("Auth changed", user);
      if (user) {
        this.userPhotoURL = user.photoURL ?? "";
        this.userInfo = `${user.displayName ?? "No Name"}`;
      }
    });
  }

  outtahere(): void {
    if (this.auth) signOut(this.auth);

    // Back to the previous page
    this.$router.back();
  }

  deleteBttn() : void{
    const auth = getAuth();
    const user = auth.currentUser;



  try{
    const auth = getAuth();
    let uid = auth.currentUser?.uid;
    const colRef = collection(db, "Users",uid +"", "locations");
    const querey =  getDocs(colRef).then((querey) => {

      querey.docs.forEach((ds :DocumentSnapshot) =>{
        const data = ds.data();       
          let ref = ds.ref;
          deleteDoc(ref);
          console.log("deleted doc");
          

       

      } );
     
      });

    

   }catch{
     console.error;

    }



    let uid = auth.currentUser?.uid;
    console.log(uid +" uid");
    //const userCol = collection(db, "Users" + ""+ uid);
    deleteDoc(doc(db,"Users", uid +"")).then(() => {console.log("Document Deleted");
     user?.delete().then(() => {
   


  this.outtahere();
  console.log("User deleted");
}).catch((error) => {
  console.log("delete Error");
});
    });

  }
}
</script>


