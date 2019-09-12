<template lang="html">
  <div name="bankList" class="banklist-container">
    <div v-if="dataLoaded && !ListBankData" class="searchSection">
      <div>
        <input class="search-field" type="text" v-model="search" placeholder="Search Banks"/>
        <select class="search-field" v-model="selected" style="height: 26px;">
          <option>MUMBAI</option>
          <option>BANGALORE</option>
          <option>DELHI</option>
          <option>HYDERABAD</option>
          <option>KOLKATA</option>
        </select>
      </div>
      <div>
        <button class="redirect-bank-btn" v-if="!showFavourites" @click="showFavourites = !showFavourites">Favourite Banks</button>
        <button class="redirect-bank-btn" v-if="showFavourites" @click="showFavourites = !showFavourites">All Banks</button>
      </div>
    </div>
    <table v-if="dataLoaded && !ListBankData" class="container">
    	<thead v-if="toggleBankData.length > 0">
    		<tr class="header-row row">
    			<th class="cell primary"><h1>Bank Name</h1></th>
    			<th class="cell"><h1>IFSC</h1></th>
    			<th class="cell"><h1>Branch</h1></th>
    			<th class="cell"><h1>City</h1></th>
          <th class="cell"><h1>Address</h1></th>
          <th class="cell mobile-hide"><h1>Fav Bank</h1></th>
    		</tr>
    	</thead>
    	<tbody>
    		<tr v-for="bank in toggleBankData" @click="getBranchData(bank)" class="row">
          <input class="mobile-show" type="radio" name="expand">
          <td class="cell primary">{{bank.bank_name}}</td>
    			<td class="cell" data-label="IFSC">{{bank.ifsc}}</td>
    			<td class="cell" data-label="Branch">{{bank.branch}}</td>
    			<td class="cell" data-label="City">{{bank.city}}</td>
          <td class="cell" data-label="Address" style="word-break: break-word;">{{bank.address}}</td>
          <td class="cell mobile-hide" style="text-align: center;" data-label="fav Bank">
            <input type="checkbox" v-model="favBanks[bank.ifsc]" :value="favBanks[bank.ifsc]" @change="makeFav(bank)" />
          </td>
    		</tr>
        <tr class="emptystate" v-if="toggleBankData.length <= 0">
          <p class="emptystate-h">No Banks Data Found on the Current Search</p>
        </tr>
    	</tbody>
    </table>
    <div v-if="dataLoaded && toggleBankData.length > 0 && !ListBankData" class="pagination">
      <span @click="setPage(currentPageIndex - 1)" class="button-before">Prev</span>
      <span v-for="i in pages" @click="setPage(i - 1)" :key="i" v-bind:class="{'current': (currentPageIndex + 1) === i}" class="pag-btn">{{i}}</span>
      <span @click="setPage(currentPageIndex + 1)" class="button-next">Next</span>
    </div>
    <ListData v-if="ListBankData && dataLoaded" :bankData="branchData" @goBack="goBack()"></ListData>
    <PreLoader v-if="!dataLoaded"></PreLoader>
  </div>
</template>

<script>
import axios from 'axios'
import PreLoader from '../components/PreLoader'
import ListData from '../components/listdata'
import $ from 'jquery'
import {_} from 'vue-underscore'

export default {
  name: 'bankList',
  components: { PreLoader, ListData },
  data: function () {
    return {
      bankdata: [],
      dataLoaded: false,
      showFavourites: false,
      ListBankData: false,
      favBanks: {},
      perpage: 4,
      branchData: {},
      filteredfavBanks: [],
      currentPageIndex: 0,
      search: '',
      selected: 'MUMBAI'
    }
  },
  watch: {
    perpage: function (newValue, oldValue) {
      this.perpage = parseInt(newValue)
      this.currentPageIndex = 0
    },
    selected: function (city) {
      this.getBankData(city)
    }
  },
  mounted () {
    this.getBankData(this.selected)
  },
  computed: {
    toggleBankData: function () {
      if (this.showFavourites === true) {
        return this.filteredfavBanks
      } else {
        return this.bankDataPerPage
      }
    },
    pages: function () {
      var filteredData = this.filteredData
      var x = Math.ceil(filteredData.length / this.perpage)
      var arr = []
      for (var i = 1; i <= x; i++) {
        arr.push(i)
      }
      if (arr.slice(this.currentPageIndex, this.currentPageIndex + 5).length < 5) {
        return arr.slice(x - 5, x + 1)
      } else {
        return arr.slice(this.currentPageIndex, this.currentPageIndex + 5)
      }
    },
    bankDataPerPage: function () {
      return this.filteredData.slice(this.currentPageIndex * this.perpage, this.currentPageIndex * this.perpage + this.perpage)
    },
    filteredData: function () {
      if (this.search !== '') {
        return this.bankdata.filter(bank => {
          var banks
          banks = bank.bank_name.toLowerCase().includes(this.search.toLowerCase())
          banks = bank.ifsc.toLowerCase().includes(this.search.toLowerCase())
          banks = bank.branch.toLowerCase().includes(this.search.toLowerCase())
          banks = bank.city.toLowerCase().includes(this.search.toLowerCase())
          banks = bank.address.toLowerCase().includes(this.search.toLowerCase())
          return banks
        })
      }
      return this.bankdata
    }
  },
  methods: {
    getBankData: function (city) {
      axios.get(`https://vast-shore-74260.herokuapp.com/banks?city=${city}`).then(r => {
        this.bankdata = r.data
        this.dataLoaded = true
        var allFavr = JSON.parse(localStorage.getItem('favBanks'))
        allFavr.map(r => {
          this.favBanks[r] = true
        })
        var filteredBanks = []
        for (let key in this.favBanks) {
          var banks = _.find(this.bankdata, (bank) => {
            return bank.ifsc == key
          })
          if (banks !== undefined) {
            filteredBanks.push(banks)
          }
        }
        this.filteredfavBanks = filteredBanks
      }).catch(err => {
        console.log(err)
      })
    },
    makeFav: function (data) {
      var allFavr = JSON.parse(localStorage.getItem('favBanks'))
      if (allFavr == undefined) {
        var allFavr = []
        localStorage.setItem('favBanks', JSON.stringify(allFavr))
        allFavr.push(data.ifsc)
        localStorage.setItem('favBanks', JSON.stringify(allFavr))
      } else {
          var index = allFavr.indexOf(data.ifsc)
          if (index !== -1) {
            allFavr.splice(index, 1)
            localStorage.setItem('favBanks', JSON.stringify(allFavr))
          } else {
            allFavr.push(data.ifsc)
            localStorage.setItem('favBanks', JSON.stringify(allFavr))
          }
        }
      },
      getBranchData: function (data) {
        this.branchData = data
        if ($(window).width() > 760) {
          this.ListBankData = true
        }
      },
      goBack: function () {
        this.ListBankData = false
      },
      setPage: function (index) {
        var length = Math.ceil(this.filteredData.length / this.perpage)
        if (index >= 0 && index < length) {
          this.currentPageIndex = index
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
</style>
