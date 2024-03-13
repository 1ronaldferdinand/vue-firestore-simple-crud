<script>
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getFirestore, onSnapshot, collection, doc, deleteDoc, setDoc, addDoc, orderBy, query } from 'firebase/firestore';
import { onMounted, ref } from 'vue';
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyCL3g9TlcIC4YOGCm4BxiNZ07rIBZgIgNg",
  authDomain: "vue-firebase-d8684.firebaseapp.com",
  projectId: "vue-firebase-d8684",
  storageBucket: "vue-firebase-d8684.appspot.com",
  messagingSenderId: "714718357899",
  appId: "1:714718357899:web:f05737e9741a9b5d1ae78e"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

export default {
  name: 'App',
  data: () => {
    return {
      books:ref([]),

      updateId: null,
      isUpdate: false,
    }
  },
  mounted() {
    const latestQuery = query(collection(db, 'books'), orderBy('name'));
    const liveBooks = onSnapshot(latestQuery, (snapshot)=> {
      this.books = snapshot.docs.map((doc) => {
        // Mendapatkan data dari dokumen Firestore
        const data = doc.data();
        const newData = {};

        // Membuat objek baru hanya dengan field dan nilai yang ada di dokumen
        Object.keys(data).forEach(key => {
          newData['id'] = doc.id;
          newData[key] = data[key];
        });
        return newData;
      });

      this.$refs.bookName.value = '';
      this.$refs.bookPrice.value = 0;
    });

    onMounted(liveBooks);
  },
  methods: {
    addNewBook:function(){
      addDoc(collection(db, 'books'), {
        name: this.$refs.bookName.value,
        price: Number(this.$refs.bookPrice.value)
      })
    },
    doUpdateBook:function(){
      setDoc(doc(db, 'books', this.updateId), {
        name: this.$refs.bookName.value,
        price: Number(this.$refs.bookPrice.value)
      })

      this.isUpdate = false;
    },
    updateBook:function(book){
      this.updateId = book.id;
      this.isUpdate = true;

      this.$refs.bookName.value = book.name;
      this.$refs.bookPrice.value = book.price;
    },
    deleteBook:function(id){
      deleteDoc(doc(db, 'books', id));
    }
  }
}
</script>

<template>
  <div id="app" style="height: 100vh; width: 100vw; display: flex; flex-direction: column;">
    <div style="display: flex; align-items: center; justify-content: flex-start; margin: 1rem 0; gap: 1rem;">
      <input ref="bookName" placeholder="add new book..." type="text">
      <input ref="bookPrice" placeholder="add new book price..." type="number">
      <button v-if="isUpdate" @click="doUpdateBook">Update Book</button>
      <button v-else @click="addNewBook">Add Book</button>
    </div>
    
    <div style="display: flex; align-items: center;" v-for="book in books" :key="book.id">
      <span style="width: 25%;">{{ book.name }}</span>
      <span style="width: 25%;">{{ book.id }}</span>
      <span style="width: 25%;">{{ book.price }}</span>
      <button @click="updateBook(book)">Update</button>
      <button @click="deleteBook(book.id)">Delete</button>
    </div>
  </div>
</template>

<style scoped>
</style>