<template>
  <div class="container">
    <div style="width: 100%;">
      <b-row>
        <b-col md="12">
          <b-button class="mb-3 float-left" variant="primary" size="sm" v-b-modal.form-person @click="resetPerson">
            Add user
          </b-button>
          <b-table hover :items="persons" :fields="fields">
            <template slot="[actions]" slot-scope="row">
              <b-button v-b-modal.form-person variant="warning" size="sm" class="mr-1" @click="editPerson(row.item)">
                Edit
              </b-button>
              <b-button v-b-modal.delete-confirmation variant="danger" size="sm" @click="deletePerson(row.item.id)">
                Delete
              </b-button>
            </template>
          </b-table>

          <b-modal id="form-person" hide-footer title="Edit user data">
            <b-row>
              <b-col md="12">
                <b-form-group>
                  <label>First Name</label>
                  <b-form-input v-model="person.firstname" placeholder="Enter your first name"></b-form-input>
                </b-form-group>
                <b-form-group>
                  <label>Last Name</label>
                  <b-form-input v-model="person.lastname" placeholder="Enter your last name"></b-form-input>
                </b-form-group>
                <b-form-group>
                  <label>Phone</label>
                  <b-form-input v-model="person.phone" placeholder="Enter your phone number"></b-form-input>
                </b-form-group>
                <b-form-group>
                  <label>Email</label>
                  <b-form-input v-model="person.email" placeholder="Enter your email"></b-form-input>
                </b-form-group>
                <b-form-group>
                  <b-button variant="primary" size="sm" @click="submit()">Submit</b-button>
                  <b-button variant="warning" size="sm" @click="$bvModal.hide('form-person')">Cancel</b-button>
                </b-form-group>
              </b-col>
            </b-row>
          </b-modal>

          <b-modal id="delete-confirmation" hide-footer title="Delete User">
            <center>
              <p class="my-4">Are you sure?</p>
              <b-button variant="warning" @click="$bvModal.hide('delete-confirmation')" size="sm">No</b-button>
              <b-button variant="primary" @click="doDelete" size="sm">Yes</b-button>
            </center>
          </b-modal>

        </b-col>
      </b-row>
    </div>
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'

export default {
  components: {
    Logo
  },
  data() {
    return {
      fields: ['firstname', 'lastname', 'phone', 'email', {key: 'actions', label: 'Aksi'}],
      persons: [],
      person: {
        firstname: '',
        lastname: '',
        phone: '',
        email: ''
      },
      submitType: '',
      personID: null
    }
  },
  mounted(){
    this.getPersons()
  },  
  methods: {
    getPersons() {
      this.$axios.get(`v1/person`, { 
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
        }
      })
      .then((response) => {
        this.persons = response.data.data.users
      })
      .catch((error) => {
        console.log(JSON.stringify(error.response))
      })
    },
    editPerson(person) {
      this.submitType = 'update'
      this.personID = person.id

      this.person = {
        firstname: person.firstname,
        lastname: person.lastname,
        phone: person.phone,
        email: person.email
      }
    },
    resetPerson() {
      this.submitType = 'create'
      this.personID = null

      this.person = {
        firstname: '',
        lastname: '',
        phone: '',
        email: ''
      }
    },
    submit() {
      switch(this.submitType) {
        case 'create':
          this.createPerson()
          break
        case 'update':
          this.updatePerson()
          break
      }
    },
    createPerson() {
      const params = new FormData()
      params.append('firstname', this.person.firstname)
      params.append('lastname', this.person.lastname)
      params.append('phone', this.person.phone)
      params.append('email', this.person.email)

      this.$axios.post(`v1/person`, params, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
        }
      }).then((response)=> {
        this.resetPerson()
        this.getPersons()
        this.$bvModal.hide('form-person')

        this.$bvToast.toast('Create User Success!', {
          title: `Create User`,
          variant: `success`,
          solid: true
        })
      }).catch((error) => {
        this.$bvToast.toast('Create User Failed!', {
          title: `Create User`,
          variant: `danger`,
          solid: true
        })
      })
    },
    updatePerson() {
      const params = new FormData()
      params.append('firstname', this.person.firstname)
      params.append('lastname', this.person.lastname)
      params.append('phone', this.person.phone)
      params.append('email', this.person.email)

      this.$axios.patch(`v1/person/`+this.personID, params, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
        }
      }).then((response) => {
        this.resetPerson()
        this.getPersons()
        this.$bvModal.hide('form-person')

        this.$bvToast.toast('Update User Success!', {
          title: `Update User`,
          variant: `success`,
          solid: true
        })
      }).catch((error) => {
        this.$bvToast.toast('Update User Failed!', {
          title: `Update User`,
          variant: `danger`,
          solid: true
        })
      })
    },
    deletePerson(personID) {
      this.personID = personID
    },
    doDelete() {
      this.$axios.delete(`v1/person/`+this.personID, {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
        }
      }).then((response) => {
        this.personID = null
        this.getPersons()
        this.$bvModal.hide('delete-confirmation')

        this.$bvToast.toast('Delete User Success!', {
          title: `Delete User`,
          variant: `success`,
          solid: true
        })
      }).catch((error) => {
        this.$bvToast.toast('Delete User Failed!', {
          title: `Delete User`,
          variant: `danger`,
          solid: true
        })
      })
    }
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
