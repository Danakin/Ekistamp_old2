<template>
  <div class="container">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <div class="card">
          <div class="card-header">Select Station</div>

          <div class="card-body">
            <form>
              <div class="form-group">
                <label for="exampleFormControlSelect1">Prefecture</label>
                <select
                  class="form-control"
                  id="exampleFormControlSelect1"
                  v-model="selectedPrefecture"
                    v-on:change="getCities"
                >
                  <option value="0"></option>
                  <option
                    v-for="prefecture in prefectures"
                    :key="prefecture.id"
                    :value="prefecture.id"
                  >{{ prefecture.kanji }}</option>
                </select>
              </div>
              <div class="form-group">
                <label for="exampleFormControlSelect1">City</label>
                <select class="form-control" id="exampleFormControlSelect1" v-model="selectedCity" v-on:change="getStations">
                  <option value="0">全て</option>
                  <option
                    v-for="city in filteredCities"
                    :key="city.id"
                    :value="city.id"
                  >{{ city.kanji }}</option>
                </select>
              </div>
              <div class="form-group">
                <label for="exampleFormControlSelect1">Station</label>
                <select
                  class="form-control"
                  id="exampleFormControlSelect1"
                  v-model="selectedStation"
                >
                  <option value="0">全て</option>
                  <option
                    v-for="station in filteredStations"
                    :key="station.id"
                    :value="station.id"
                  >{{ station.kanji }}</option>
                </select>
              </div>
            </form>
            <button type="button" class="btn btn-primary" v-on:click="addStation">Add Station</button>
          </div>
        </div>
      </div>

      <div class="col-md-8">
        <div class="card">
          <div class="card-header">Stations to Add</div>
          <div class="card-body">
            <table class="table">
              <thead>
                <th scope="col">id</th>
                <th scope="col">Prefecture</th>
                <th scope="col">City</th>
                <th scope="col">Station</th>
              </thead>
              <tbody>
                <tr v-for="station in newStations" :key="station.id">
                  <td scope="row">{{ station.id }}</td>
                  <td>{{ station.p_kanji }}</td>
                  <td>{{ station.c_kanji }}</td>
                  <td>{{ station.s_kanji }}</td>
                </tr>
              </tbody>
            </table>
            <button type="button" class="btn btn-primary" v-on:click="submit">Add Station</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
const uri = "http://127.0.0.1:8000/"

export default {
  data() {
    return {
      selectedPrefecture: 0,
      selectedCity: 0,
      selectedStation: 0,
      addedStations: [],
      prefectures: [],
      cities: [],
      stations: []
    };
  },
  mounted() {
      this.getPrefectures()
  },
  computed: {
    filteredCities: function() {
      const selectedIndex = Number(this.selectedPrefecture);
      if (selectedIndex === 0 || selectedIndex > 47) {
        return this.cities;
      } else {
        return this.cities.filter(city => city.prefecture_id === selectedIndex);
      }
    },
    filteredStations: function() {
      const selectedPrefecture = Number(this.selectedPrefecture);
      const selectedCity = Number(this.selectedCity);
      if (selectedPrefecture > 0 && selectedCity > 0) {
        return this.stations.filter(
          station =>
            station.prefecture_id === selectedPrefecture &&
            station.city_id === selectedCity
        );
      } else if (selectedPrefecture > 0) {
        return this.stations.filter(
          station => station.prefecture_id === selectedPrefecture
        );
      } else if (selectedCity > 0) {
        return this.stations.filter(
          station => station.city_id === selectedCity
        );
      } else {
        return this.stations;
      }
    },
    newStations: function() {
      return this.stations
        .filter(station => this.addedStations.includes(station.id))
        .map(station => {
          const prefecture = this.prefectures.filter(
            prefecture => prefecture.id === station.prefecture_id
          )[0];
          const city = this.cities.filter(
            city => city.id === station.city_id
          )[0];
          return {
            id: station.id,
            p_kanji: prefecture.kanji,
            p_kana: prefecture.kana,
            p_romaji: prefecture.romaji,
            c_kanji: city.kanji,
            c_kana: city.kana,
            c_romaji: city.romaji,
            s_kanji: station.kanji,
            s_kana: station.kana,
            s_romaji: station.romaji
          };
        });
    }
  },
  methods: {
    getPrefectures: function() {
        if(this.prefectures.length === 0) {
            axios.get(uri + "prefectures")
            .then(response => {
                if(response.status === 200) {
                    this.prefectures = response.data;
                }
            })
            .catch(error => {
                console.error(error)
            })
        }
    },
    getCities: function() {
        if(this.filteredCities.length === 0) {
            axios.get(uri + 'prefectures/c/' + this.selectedPrefecture)
            .then(response => {
                this.cities = this.cities.concat(response.data);
            })
            .catch(error => {
                console.error(error);
            })
        }
    },
    getStations: function() {
        if(this.filteredStations.length === 0) {
            axios.get(uri + 'cities/s/' + this.selectedCity)
            .then(response => {
                this.stations = this.stations.concat(response.data);
            })
            .catch(error => {
                console.error(error);
            })
        }
    },
    resetCity: function() {
      this.selectedCity = 0;
      this.resetStation();
    },
    resetStation: function() {
      this.selectedStation = 0;
    },
    addStation: function(e) {
      if (
        this.selectedStation > 0 &&
        !this.addedStations.includes(this.selectedStation)
      ) {
        this.addedStations.push(this.selectedStation);
      }
    },
    submit: function() {
        axios.post(
            "/us",
            {data: this.addedStations}
        )
        .then(response => {
            console.log(response);
        });
    }
  }
};
</script>
