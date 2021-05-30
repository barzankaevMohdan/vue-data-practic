<template>
  <div class="container">
    <app-allert
    :alert="alert"
    @close="alert = null"
    ></app-allert>
    <form class="card" @submit.prevent="createPerson">
      <h2>Работа с базой данных</h2>

      <div class="form-control">
        <label for="name">Введите имя</label>
        <input type="text" id="name" v-model.trim="name">
      </div>

      <button class="btn primary" :disabled="name.length === 0">Create</button>
    </form>

    <app-people-list
    :people="people"
    @load="loadPeople"
    @remove="removePerson"
    ></app-people-list>
  </div>
</template>

<script>
import AppPeopleList from './components/AppPeopleList'
import appAllert from './components/AppAlert'

export default {
  data() {
    return {
      name: '',
      people: [],
      alert: null
    }
  },
  mounted() {
    this.loadPeople()
  },
  methods: {
    async createPerson() {
      const response = await fetch('https://vue-data-284a7-default-rtdb.firebaseio.com/people.json',{
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          firstName: this.name
        })
      })
      const firebaseData = await response.json()

      this.people.push({
        firstName: this.name,
        id: firebaseData.name
      })
      this.name = ''
    },
    async loadPeople() {
      try{
        const response = await fetch ('https://vue-data-284a7-default-rtdb.firebaseio.com/people.json',{
        method: 'GET'
        })
        const firebaseData = await response.json()
        if(!firebaseData) {
          throw new Error('Список людей пуст')
        }
        this.people = Object.keys(firebaseData).map(key => {
          return {
            id: key,
            ...firebaseData[key]
          }
      })
      } catch (e) {
        this.alert = {
          type: 'danger',
          title: 'Ошибка',
          text: e.message
        }
      }
    },
    async removePerson(id) {
      const response = await fetch (`https://vue-data-284a7-default-rtdb.firebaseio.com/people/${id}.json`,{
        method: 'DELETE'
      })
      await response.json()
      this.people = this.people.filter(person => person.id !== id)
    }
  },
  components: {AppPeopleList, appAllert}
}
</script>

<style>

</style>
