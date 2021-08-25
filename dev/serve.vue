<script>
import Vue from 'vue';
import VueDataGrid from '@/vue-data-grid.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueDataGrid
  },

  data() {
    return {
      users: null
    }
  },
  computed: {
    setColumns() {
      return [
        { key: 'id', value: 'id', sort: false, filter: {type: 'number', onNumber: '', beforeNumber: '', afterNumber: ''} },
        { key: 'first_name', value: 'first_name', sort: false, filter: {type: 'list', placeholder: 'Rechercher', options: this.users.map(user => user.first_name), values: ''} },
        { key: 'last_name', value: 'last_name', sort: false, filter: {type: 'list', placeholder: 'Rechercher', options: this.users.map(user => user.last_name), values: ''} },
        { key: 'email', value: 'email', sort: false, filter: {type: 'list', placeholder: 'Rechercher', options: this.users.map(user => user.email), values: ''} },
        { key: 'gender', value: 'gender', sort: false, filter: {type: 'list', placeholder: 'Rechercher', options: this.users.map(user => user.gender), values: ''} },
        { key: 'social_insurance_number', value: 'social_insurance_number', sort: false, filter: {type: 'number', onNumber: '', beforeNumber: '', afterNumber: ''} },
        { key: 'date_of_birth', value: 'date_of_birth', sort: false, filter: {type: 'date', onDate: '', beforeDate: '', afterDate: ''} },
        { key: 'password', value: 'password', sort: false, filter: {type: 'list', placeholder: 'Rechercher', options: this.users.map(user => user.password), values: ''} },
      ]
    }
  },
  mounted() {
    fetch('https://random-data-api.com/api/users/random_user?size=100')
      .then(res => res.json())
      .then(data => this.users = data)
  }
});
</script>

<template>
  <div id="app">
    <VueDataGrid
      v-if="users"
      :datas="users"
      :columns="setColumns"
      :page_limit="25"
      :checkbox="true"
      :stickyThead="true"
      :stickyCol="true"
      :action_col="true"
      :searchbar="true"
      v-slot="slotProps"
    >
      <td :style="slotProps.setWidthColumn('220px')" :class="{'sticky': slotProps.stickyCol}">{{slotProps.row.id}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.first_name}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.last_name}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.email}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.gender}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.social_insurance_number}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.date_of_birth}}</td>
      <td :style="slotProps.setWidthColumn('220px')">{{slotProps.row.password}}</td>
      <td class="quick-actions">
        <button :id="`openDropdown-${slotProps.index}`" class="cmg-btn cmg-more-actions" @click="$openDropdown(`#openDropdown-${slotProps.index}`)">
          <!-- <MoreActions /> -->
          <div class="cmg-dropdown left">
            <div>
              <a @click="fillOverlay(slotProps.row.id, slotProps.row.housing_id)">
                <span class="list-text">Voir la réservation</span>
              </a>
            </div>
          </div>
        </button>
      </td>
    </VueDataGrid>
  </div>
</template>
