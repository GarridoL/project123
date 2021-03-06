<template>
  <div class="ledger-summary-container">
    <div class="incre-row">
      <button class="btn btn-primary pull-right" @click="showModal('create')">Add</button>
    </div>
    <basic-filter 
      v-bind:category="category" 
      :activeCategoryIndex="0"
      :activeSortingIndex="0"
      @changeSortEvent="retrieve($event.sort, $event.filter)"
      @changeStyle="manageGrid($event)"
      :grid="['list', 'th-large']"></basic-filter>
    
    <table class="table table-bordered table-responsive" v-if="data !== null">
      <thead>
        <tr>
          <td>Country</td>
          <td>Currency</td>
          <td>Code</td>
          <td>Locality</td>
          <td>Type</td>
          <td>Amount</td>
          <td>Limit</td>
          <td>Start</td>
          <td>End</td>
          <td>Actions</td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in data" :key="index">
          <td>{{item.country}}</td>
          <td>{{item.currency}}</td>
          <td>{{item.code}}</td>
          <td>{{item.locality}}</td>
          <td>{{item.type}}</td>
          <td>{{item.amount}}</td>
          <td>{{item.limit}}</td>
          <td>{{item.start}}</td>
          <td>{{item.end}}</td>
          <td>
            <button class="btn btn-primary" @click="showModal('update', item)"><i class="fa fa-edit"></i></button>
            <button class="btn btn-danger" @click="remove(item.id)"><i class="fa fa-trash"></i></button>
          </td>
        </tr>
      </tbody>
    </table>
    <Pager
      :pages="numPages"
      :active="activePage"
      :limit="limit"
      v-if="data !== null"
    />
    <empty v-if="data === null" :title="'No coupons added!'" :action="'Click add to create.'"></empty>
    <browse-images-modal></browse-images-modal>
    <increment-modal :property="modalProperty"></increment-modal>
  </div>
</template>
<style scoped>
.ledger-summary-container{
  width: 100%;
  float: left;
  height: auto;
  margin-bottom: 100px;
  margin-top: 25px;
}

.ledger-summary-container-header{
  width: 100%;
  float: left;
  height: 70px;
  border: solid 1px #ddd;
}
.summary-container-item{
  width: 100%;
  float: left;
  border-radius: 5px;
  min-height: 50px;
  overflow-y: hidden;
  border: solid 1px #ddd;
  margin-top: 10px;
  padding-left: 10px;
}
.summary-container-item .header{
  width: 100%;
  float: left;
  height: 50px;
  line-height: 50px;
  color: #555;
}
.summary-container-item .body{
  width: 100%;
  float: left;
  min-height: 50px;
  overflow-y: hidden;
  padding-right: 10px;
}

@media (max-width: 992px){
  .ledger-summary-container{
    width: 100%;
  }
}
</style>
<script>
import ROUTER from 'src/router'
import AUTH from 'src/services/auth'
import CONFIG from 'src/config.js'
import ModalProperty from 'src/modules/admin/CreateCoupons.js'
import Pager from 'src/components/increment/generic/pager/Pager.vue'
export default{
  mounted(){
    $('#loading').css({display: 'block'})
    this.retrieve({created_at: 'desc'}, {column: 'created_at', value: ''})
  },
  data(){
    return {
      user: AUTH.user,
      data: null,
      auth: AUTH,
      newAttachment: {
        activeId: null,
        file: null
      },
      config: CONFIG,
      modalProperty: ModalProperty,
      category: [{
        title: 'Sort by',
        sorting: [{
          title: 'Date posted ascending',
          payload: 'created_at',
          payload_value: 'asc'
        }, {
          title: 'Date posted descending',
          payload: 'created_at',
          payload_value: 'desc'
        }, {
          title: 'Type ascending',
          payload: 'type',
          payload_value: 'asc'
        }, {
          title: 'Type descending',
          payload: 'type',
          payload_value: 'desc'
        }, {
          title: 'Amount ascending',
          payload: 'amount',
          payload_value: 'asc'
        }, {
          title: 'Amount descending',
          payload: 'amount',
          payload_value: 'desc'
        }, {
          title: 'Country ascending',
          payload: 'country',
          payload_value: 'asc'
        }, {
          title: 'Country descending',
          payload: 'country',
          payload_value: 'desc'
        }, {
          title: 'Locality ascending',
          payload: 'locality',
          payload_value: 'asc'
        }, {
          title: 'Locality descending',
          payload: 'locality',
          payload_value: 'desc'
        }, {
          title: 'Start ascending',
          payload: 'start',
          payload_value: 'asc'
        }, {
          title: 'Start descending',
          payload: 'start',
          payload_value: 'desc'
        }, {
          title: 'End ascending',
          payload: 'end',
          payload_value: 'asc'
        }, {
          title: 'End descending',
          payload: 'end',
          payload_value: 'desc'
        }]
      }],
      currentFilter: null,
      currentSort: null,
      activePage: 1,
      numPages: null,
      limit: 5
    }
  },
  components: {
    'empty': require('components/increment/generic/empty/Empty.vue'),
    'browse-images-modal': require('components/increment/generic/image/BrowseModal.vue'),
    'basic-filter': require('components/increment/generic/filter/Basic.vue'),
    'increment-modal': require('components/increment/generic/modal/Modal.vue'),
    Pager
  },
  methods: {
    redirect(params){
      ROUTER.push(params)
    },
    retrieve(sort, filter){
      if(sort !== null){
        this.currentSort = sort
      }
      if(filter !== null){
        this.currentFilter = filter
      }
      let parameter = {
        condition: [{
          column: this.currentFilter.column,
          clause: 'like',
          value: '%' + this.currentFilter.value + '%'
        }],
        sort: this.currentSort,
        limit: this.limit,
        offset: (this.activePage > 0) ? ((this.activePage - 1) * this.limit) : this.activePage
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('coupons/retrieve', parameter).then(response => {
        $('#loading').css({display: 'none'})
        console.log(response)
        if(response.data.length > 0){
          this.data = response.data
          this.numPages = parseInt(response.size / this.limit) + (response.size % this.limit) ? 1 : 0
        }else{
          this.data = null
          this.numPages = null
        }
      })
    },
    remove(id){
      let parameter = {
        id: id
      }
      $('#loading').css({display: 'block'})
      this.APIRequest('coupons/delete', parameter).then(response => {
        this._retrieve({type: 'asc'}, {column: 'created_at', value: ''})
      })
    },
    showModal(action, item = null){
      switch(action){
        case 'create':
          this.modalProperty = {...ModalProperty}
          let inputs = this.modalProperty.inputs
          inputs.map(input => {
            input.value = null
          })
          this.modalProperty.params[0].value = this.user.userID
          break
        case 'update':
          let modalData = {...this.modalProperty}
          let parameter = {
            title: 'Update Requests',
            route: 'coupons/update',
            button: {
              left: 'Cancel',
              right: 'Update'
            },
            sort: {
              column: 'created_at',
              value: 'desc'
            },
            params: [{
              variable: 'id',
              value: item.id
            }]
          }
          modalData = {...modalData, ...parameter} // updated data without
          let object = Object.keys(item)
          modalData.inputs.map(data => {
            if(data.variable === 'code'){
              data.value = item.code
            }
            if(data.variable === 'currency'){
              data.value = item.currency
            }
            if(data.variable === 'country'){
              data.value = item.country
            }
            if(data.variable === 'locality'){
              data.value = item.locality
            }
            if(data.variable === 'type'){
              data.value = item.type
            }
            if(data.variable === 'amount'){
              data.value = item.amount
            }
            if(data.variable === 'limit'){
              data.value = item.limit
            }
            if(data.variable === 'start'){
              data.value = item.start
            }
            if(data.variable === 'end'){
              data.value = item.end
            }
          })
          this.modalProperty = {...modalData}
          break
      }
      $('#createCouponsModal').modal('show')
    },
    manageGrid(event){
      switch(event){
        case 'th-large': this.listStyle = 'columns'
          break
        case 'list': this.listStyle = 'list'
          break
      }
    }
  }
}
</script>
