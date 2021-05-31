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

    <app-loader v-if="loading"></app-loader>

    <app-people-list
    v-else
    :people="people"
    @load="loadPeople"
    @remove="removePerson"
    ></app-people-list>
  </div>
</template>

<script>
import AppPeopleList from './components/AppPeopleList'
import AppAllert from './components/AppAlert'
import AppLoader from './components/AppLoader'

export default {
  data() {
    return {
      name: '',
      people: [],
      alert: null,
      loading: false
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
        this.loading = true
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
      this.loading = false
      } catch (e) {
        this.alert = {
          type: 'danger',
          title: 'Ошибка',
          text: e.message
        }
        this.loading = false
      }
    },
    async removePerson(id) {
      try {
        const name = this.people.find((person) => person.id === id).firstName
        const response = await fetch (`https://vue-data-284a7-default-rtdb.firebaseio.com/people/${id}.json`,{
        method: 'DELETE'
        })
        await response.json()
        this.people = this.people.filter(person => person.id !== id)
        this.alert = {
          type: 'primary',
          title: 'Успешно',
          text: `Пользователь "${name}" успешно удален.`
        }
      } catch (e) {

      }
    }
  },
  components: {AppPeopleList, AppAllert, AppLoader}
}
</script>

<style>

</style>
