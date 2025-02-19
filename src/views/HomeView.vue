<script setup>
  import CreateListing from '../components/CreateListing.vue';
  import CarCardDisplay from '../components/CarCardDisplay.vue';
  import KeywordSearch from '../components/KeywordSearch.vue';

  import { ref, onMounted, watch } from 'vue'
  import axios from 'axios'

  const props = defineProps({
    userId: {
      type: Number,
      required: true
    }
  })
  
  const listOfCars = ref([])
  const filteredCars = ref([])
  const listOfCompanies = ref([])
  const nextId = ref(0)
  const init = onMounted(async () => {
    await axios
    .get('http://localhost:8085/cars')
    .then(response => {listOfCars.value = response.data, 
      filteredCars.value = response.data, 
      nextId.value = (Object.keys(response.data).length > 0) ? 
      response.data[Object.keys(response.data).length - 1].id + 1 : 1})
    await axios
    .get('http://localhost:8085/company')
    .then(reponse => listOfCompanies.value = reponse.data)
  })
  
  async function deleteCar(id){
    if (confirm('Are you sure?')){
      console.log("Making a delete request")
      await axios
      .delete('http://localhost:8085/cars/' + id)
      .then(response => {console.log('Delete successfull!')})
      listOfCars.value = listOfCars.value.filter((car) => car.id !== id)
      init()
    }
  }

  async function addCar(form, nextId){
    console.log("Making a create request")
    await axios
    .post('http://localhost:8085/cars', {
      "id": nextId,
      "make": form.make,
      "sellerId": 1,
      "model": form.model,
      "releaseYear": form.releaseYear,
      "fuelType": form.fuelType,
      "price": form.price,
      "vehicleType": form.vehicleType,
      "hp": form.hp,
      "mileage": form.mileage,
      "colour": form.colour,
      "transmission": form.transmission,
      'carURL': form.carURL,
    })
    .then(response => {[...listOfCars.value, response.data]})
    .catch(function (error) {
      console.log(error)
    })
    console.log("Create request successful")
    init()
  }

  async function updateCar(form){
    console.log(form)
    console.log("Making an update request")
    await axios
    .put('http://localhost:8085/cars/' + form.id, {
      "id": form.id,
      "make": form.make,
      "sellerId": 1,
      "model": form.model,
      "releaseYear": form.releaseYear,
      "fuelType": form.fuelType,
      "price": form.price,
      "vehicleType": form.vehicleType,
      "hp": form.hp,
      "mileage": form.mileage,
      "colour": form.colour,
      "transmission": form.transmission,
      'carURL': form.carURL,
    })
    .then(response => {[...listOfCars.value, response.data]})
    .catch(function (error) {
      console.log(error)
    })
    console.log("Update request successful")
    init()
  }

  async function searchCar(searchResults){
      console.log("Making a search request")
      filteredCars.value = searchResults.value
  }
 

  const companyFilter = ref([])
  
  watch(companyFilter, () => {
    console.log(companyFilter.value.length)
    if (companyFilter.value.length == 0) {
      filteredCars.value = listOfCars.value 
    }
    else{
      filteredCars.value = listOfCars.value.filter(car => companyFilter.value.includes(car.company.make))
    }
  })
</script>

<template>
  <main>
    <div class="container py-4">
      <div class="row g-4 justify-content-center">
        <div class="col-md-12 col-lg-10 text-center">
          <KeywordSearch
          :listOfCars="listOfCars"
          @search-car="searchCar"/>
        </div>

        <!-- Button to trigger create modal -->
        <div class="col-md-12 col-lg-2 text-center">
          <button type="button" class="btn btn-success" data-bs-toggle="modal" data-bs-target="#createModal">
            Create Vehicle Listing 
          </button>
        </div>
      </div>
      
      <div class="row py-4">
        <div class="col-lg-2 col-md-12 col-sm-12 filters">
          <div class="card border-0" style="width: 18rem;">
            <div class="card-body">
              <h5 class="card-title">Filters</h5>
              <h6 class="card-subtitle mb-2 text-muted">Find the perfect car!</h6>
              <div class="dropdown">
                <a class="btn btn-primary-outline dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                  Companies
                </a>
                <ul class="dropdown-menu">
                  <li v-for="company in listOfCompanies" v-bind:key="company.make">
                    <div class="form-check m-4">
                      <input class="form-check-input" type="checkbox" :id="company.make" :value="company.make" v-model="companyFilter">
                      <label class="form-check-label" :for="company.make">
                        {{ company.make }}
                      </label>
                    </div>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>

        <div class="col-lg-10 col-md-12 col-sm-12">
          <CarCardDisplay @delete-car="deleteCar" 
          @update-car="updateCar"  
          :cars="filteredCars" 
          :userId="props.userId"/>
        </div>
      </div>
    </div>
    <CreateListing 
    @add-car="addCar"
    :nextId = "nextId"/>
  </main>
</template>

<style scoped>
  @import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.2/font/bootstrap-icons.css");

  .dropdown-menu{
    overflow: hidden;
    overflow-y: auto;
    max-height: 25vh;
  }
  
</style>
    

