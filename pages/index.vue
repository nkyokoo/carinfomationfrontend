<template>
  <div>
    <v-row justify="center" align="center">
      <v-col>

      </v-col>
    </v-row>
    <v-row justify="center" align="center">
      <v-col>
        <v-card>
          <v-card-text>
            <v-text-field
              type="text"
              rounded
              v-model="plateNumber"
              solo
              label="Indtast nummerplade"
              style="margin-top: 2rem"
              v-mask="'AA#####'"
              :rules="NumberPlateRules"
              @keypress.enter="getCarInfo"
            >

            </v-text-field>
          </v-card-text>
          <v-card-actions>

          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
    <v-row justify="center" align="center">
      <v-col>
        <v-card v-if="Object.keys(this.carInfo).length !== 0">
          <v-card-title>{{
              this.plateNumber
            }}
          </v-card-title>
          <v-card-text>
            <v-list
              style="max-height: 30rem"
              class="overflow-y-auto"
            >
              <v-list-item v-for="(key,i) in this.carInfoKeys" :key="i">
                <v-list-item-title>{{ formatKey(key) }}</v-list-item-title>
                <v-list-item-subtitle>{{ formatEngine(key, carInfo[key]) }}</v-list-item-subtitle>
                <v-divider></v-divider>
              </v-list-item>
            </v-list>
          </v-card-text>
          <v-card-actions>

          </v-card-actions>
          <v-expansion-panels>
            <v-expansion-panel @click="getCarInfoEnvironment">
              <v-expansion-panel-header>
                Miljø
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-list
                  style="max-height: 30rem"
                  class="overflow-y-auto"
                >
                  <v-list-item v-for="(key,i) in this.carInfoEnvironmentKeys" :key="i">
                    <v-list-item-title>{{ formatKey(key) }}</v-list-item-title>
                    <v-list-item-subtitle>{{ formatEngine(key, carInfoEnvironment[key]) }}</v-list-item-subtitle>
                    <v-divider></v-divider>
                  </v-list-item>
                </v-list>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>

        </v-card>
     <v-card v-if="Object.keys(carInfoRes).length !== 0 && carInfoRes.status === 404">
          <v-card-title>Ingen data!</v-card-title>
          <v-card-text>
            Nummerpladen eksisterer ikke
          </v-card-text>
        </v-card>
        <v-card v-if="Object.keys(carInfoRes).length === 0">
          <v-card-title>Ingen data!</v-card-title>
          <v-card-text>
            Skriv en nummerplade i tekstfeltet og tryk enter
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import {format} from 'date-fns'
import {da} from 'date-fns/locale'

export default {
  data() {
    return {
      NumberPlateRules: [
        v => !!v || "Nummerplade felt kan ikke være tomt!",
        v => /^[A-Z0-9]/.test(v) || "kun nummer og bogstaver!"
      ],
      plateNumber: "",
      carInfo: {},
      carInfoKeys: [],
      carInfoRes: {},
      carInfoEnvironment: {},
      carInfoEnvironmentKeys: [],
      carInfoEnvironmentRes: {}
    }
  },
  methods: {
    async getCarInfo() {
      try {
        const res = await this.$axios.get(`/${this.plateNumber}`)
        this.carInfo = res.data
        this.carInfoRes = res
        this.carInfoKeys = Object.keys(res.data)
        console.log(res.data['status'])
      } catch (e) {
        this.carInfoRes = e.response
        this.carInfo = {}
      }
    },
    async getCarInfoEnvironment() {
      try {
        const res = await this.$axios.get(`/${this.plateNumber}/environment`)
        console.log(res)
        this.carInfoEnvironment = res.data
        this.carInfoEnvironmentRes = res
        this.carInfoEnvironmentKeys = Object.keys(res.data)
        console.log(res.data['status'])
      } catch (e) {
        console.log(e)
      }
    },
    Capitalize(s) {
      if (typeof s !== 'string') return ' '
      return s.charAt(0).toUpperCase() + s.slice(1)
    },
    formatKey(key) {
      let newKey = key.replaceAll('_', ' ')
      return this.Capitalize(newKey)
    },
    formatEngine(key, value) {
      switch (key) {
        case 'status_date':
          console.log(key)
          console.log(value)
          return format(new Date(value), 'dd-MM-yyyy hh:mm', {locale: da})
        default:
          return !value ? "UOPLYST" : value
      }
    }
  }

}
</script>
