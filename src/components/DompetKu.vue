<template>
  <v-app>
    <v-container>
      <v-card class="pa-5" outlined>
        <v-card-title class="text-h4">Dompetku v2 – Expense & Income Management</v-card-title>
        <v-divider></v-divider>

        <v-tabs v-model="activeTab" class="mt-3">
          <v-tab>Input Data</v-tab>
          <v-tab>Rekap Data</v-tab>
          <v-tab>Statistik</v-tab>
        </v-tabs>

        <v-tabs-items v-model="activeTab">
          <!-- Input Data Tab -->
          <v-tab-item>
            <v-form @submit.prevent="addEntry">
              <v-row>
                <v-col cols="12" md="6">
                  <v-text-field
                    v-model="newEntry.description"
                    label="Keterangan"
                    outlined
                    required
                  ></v-text-field>
                </v-col>
                <v-col cols="12" md="6">
                  <v-text-field
                    v-model.number="newEntry.amount"
                    label="Jumlah Uang"
                    type="number"
                    outlined
                    required
                  ></v-text-field>
                </v-col>
                <v-col cols="12" md="6">
                  <v-select
                    v-model="newEntry.category"
                    :items="categories"
                    label="Kategori"
                    outlined
                    required
                  ></v-select>
                </v-col>
                <v-col cols="12" md="6" class="d-flex align-center">
                  <v-btn type="submit" color="primary" class="ml-auto">Submit</v-btn>
                </v-col>
              </v-row>
            </v-form>
          </v-tab-item>

          <!-- Rekap Data Tab -->
          <v-tab-item>
            <v-card-subtitle class="mt-3">
              <strong>Summary:</strong><br />
              Total Entries: {{ entries.length }}<br />
              Total Income: Rp {{ totalIncome }}<br />
              Total Expense: Rp {{ totalExpense }}<br />
              Balance: Rp {{ balance }}
            </v-card-subtitle>

            <v-list class="mt-5">
              <v-subheader>Rekap Keuangan</v-subheader>
              <v-alert v-if="entries.length === 0" type="info">Belum ada data.</v-alert>
              <v-list-item v-for="(entry, index) in entries" :key="index" class="mb-2">
                <v-list-item-content>
                  <v-list-item-title>
                    {{ entry.description }} - Rp {{ entry.amount }} ({{ entry.category }})
                  </v-list-item-title>
                </v-list-item-content>
                <v-list-item-action>
                  <v-btn icon color="red" @click="removeEntry(index)">
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </v-list-item-action>
              </v-list-item>
            </v-list>

            <v-btn color="error" class="mt-5" @click="confirmClearAll"> Hapus Semua Data </v-btn>
          </v-tab-item>

          <!-- Statistik Tab -->
          <v-tab-item>
            <v-row class="mt-5">
              <v-col cols="12" md="6">
                <canvas id="pieChart"></canvas>
              </v-col>
            </v-row>
          </v-tab-item>
        </v-tabs-items>
      </v-card>
    </v-container>
  </v-app>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import Chart from 'chart.js/auto'
import { loadEntries, saveEntries, clearEntries } from '../stores/store.js'

export default {
  data() {
    return {
      activeTab: 0,
      newEntry: {
        description: '',
        amount: null,
        category: '',
      },
      categories: ['Food', 'Transport', 'Shopping', 'Other'],
      entries: loadEntries(), // Load entries from storage on initialization
      pieChart: null,
    }
  },
  computed: {
    totalIncome() {
      return this.entries
        .filter((entry) => entry.amount > 0)
        .reduce((sum, entry) => sum + entry.amount, 0)
    },
    totalExpense() {
      return this.entries
        .filter((entry) => entry.amount < 0)
        .reduce((sum, entry) => sum + Math.abs(entry.amount), 0)
    },
    balance() {
      return this.totalIncome - this.totalExpense
    },
  },
  methods: {
    addEntry() {
      if (this.newEntry.description && this.newEntry.amount && this.newEntry.category) {
        this.entries.push({ ...this.newEntry })
        saveEntries(this.entries) // Save entries after adding a new one
        this.newEntry = { description: '', amount: null, category: '' }
        this.updateChart()
      } else {
        alert('Harap isi keterangan, jumlah uang, dan kategori.')
      }
    },
    removeEntry(index) {
      this.entries.splice(index, 1)
      saveEntries(this.entries) // Save entries after removing one
      this.updateChart()
    },
    confirmClearAll() {
      if (confirm('Apakah Anda yakin ingin menghapus semua data?')) {
        this.clearAllEntries()
      }
    },
    clearAllEntries() {
      this.entries = []
      clearEntries() // Use the clearEntries function
      this.updateChart()
    },
    updateChart() {
      if (this.pieChart) this.pieChart.destroy()

      const data = {
        labels: ['Income', 'Expense'],
        datasets: [
          {
            data: [this.totalIncome, this.totalExpense],
            backgroundColor: ['#4caf50', '#f44336'],
          },
        ],
      }

      const ctx = document.getElementById('pieChart').getContext('2d')
      this.pieChart = new Chart(ctx, {
        type: 'pie',
        data,
      })
    },
  },
  mounted() {
    this.entries = loadEntries() // Load entries when component is mounted
    this.updateChart()
  },
}
</script>

<style>
/* Custom styles for better design */
</style>
