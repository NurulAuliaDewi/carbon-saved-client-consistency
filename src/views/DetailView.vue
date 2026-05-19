<template>
  <div>
    <nav class="navbar">
      <div class="container-banner">
        <img src="../assets/logob2w.png" alt="Logo">
      </div>
      <div class="container-fluid">
        <button @click="goBackToDashboard" class="navbar-back-btn">
          <i class="fas fa-arrow-left"></i>
        </button>
        <span class="navbar-brand mb-0 h2 text-white">ATHLETE DETAIL</span>
      </div>
    </nav>
    
    <div class="content m-4">
      <div class="section-header">
        <h4 class="text-primary fw-bold mb-4">Detail Information {{ athleteName }}</h4>
        <div class="period-container">
          <div class="period-filters">
            <button 
              v-for="p in periods" 
              :key="p"
              @click="updatePeriod(p)"
              :class="['period-btn', { active: period === p }]"
            >
              {{ periodLabels[p] }}
            </button>
          </div>
          <div v-if="period === 'month'" class="dropdown-container">
            <select 
              v-model="selectedMonth" 
              @change="onMonthChange"
              class="period-dropdown"
            >
              <option v-for="month in months" :key="month.value" :value="month.value">
                {{ month.label }}
              </option>
            </select>
          </div>
          <div v-if="period === 'year'" class="dropdown-container">
            <select 
              v-model="selectedYear" 
              @change="onYearChange"
              class="period-dropdown"
            >
              <option v-for="year in years" :key="year" :value="year">
                {{ year }}
              </option>
            </select>
          </div>
          <div v-if="period === 'month'" class="dropdown-container">
            <select 
              v-model="selectedYear" 
              @change="onYearChange"
              class="period-dropdown"
            >
              <option v-for="year in years" :key="year" :value="year">
                {{ year }}
              </option>
            </select>
          </div>
        </div>
      </div>
      <div class="row g-4 mb-5">
        <div v-for="card in summaryCards" :key="card.id" class="col-md-4">
          <div class="modern-card">
            <div class="card-icon-section" :style="{ backgroundColor: card.color }">
              <img :src="card.icon" :alt="card.alt" class="card-icon">
            </div>
            <div class="card-content">
              <h6 class="card-title">{{ card.title }}</h6>
              <h3 class="card-value">{{ card.value }}</h3>
            </div>
          </div>
        </div>
      </div>
      <div v-if="isLoading" class="alert alert-info modern-alert">
        <i class="fas fa-spinner fa-spin me-2"></i>Loading activities...
      </div>
      <div v-if="error" class="alert alert-danger modern-alert">
        <i class="fas fa-exclamation-triangle me-2"></i>{{ error }}
      </div>
      <div v-if="!isLoading && activities.length === 0" class="alert alert-warning modern-alert">
        <i class="fas fa-info-circle me-2"></i>No activities found for this athlete.
      </div>
      <div v-if="activities.length > 0" class="section-header">
        <h4 class="text-primary fw-bold mb-4">Activity History</h4>
      </div>
      <div v-if="activities.length > 0" class="table-container">
        <div class="table-wrapper">
          <table class="modern-table">
            <thead>
              <tr>
                <th>No</th>
                <th>Commuter</th>
                <th>Route Name</th>
                <th>Distance (km)</th>
                <th>Moving Time</th>
                <th>Activity Date</th>
                <th>Elevation Gain</th>
                <th>Carbon Saving (kg CO<sub>2</sub>)</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="(activity, index) in activities" 
                :key="activity.id"
                class="table-row"
              >
                <td>{{ index + 1 }}</td>
                <td class="commuter-name" :title="`${activity.athlete_firstname} ${activity.athlete_lastname}`">
                  {{ activity.athlete_firstname }} {{ activity.athlete_lastname }}
                </td>
                <td class="route-name" :title="activity.activity_name">{{ activity.activity_name }}</td>
                <td>{{ formatDistance(activity.distance) }}</td>
                <td>{{ formatTime(activity.moving_time) }}</td>
                <td>{{ new Date(activity.datetime).toDateString() }}</td>
                <td>{{ activity.total_elevation_gain }}</td>
                <td class="carbon-value">{{ formatCarbonSaving(activity.carbon_saving) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const API_BASE_URL = 'https://consistency.bike2work.id/api';
// const API_BASE_URL = 'http://localhost:5001/api';

export default {
  name: "DetailView",
  
  data() {
    return {
      activities: [],
      isLoading: false,
      error: null,
      totals: { distance: 0, time: 0, carbon: 0, activity: 0 },
      period: 'day',
      selectedMonth: new Date().getMonth() + 1,
      selectedYear: new Date().getFullYear(), 
      athleteName: '',
      periods: ['day', 'month', 'year'],
      availableMonths: [],
      availableYears: [],
      allMonths: [
        { value: 1, label: 'January' },
        { value: 2, label: 'February' },
        { value: 3, label: 'March' },
        { value: 4, label: 'April' },
        { value: 5, label: 'May' },
        { value: 6, label: 'June' },
        { value: 7, label: 'July' },
        { value: 8, label: 'August' },
        { value: 9, label: 'September' },
        { value: 10, label: 'October' },
        { value: 11, label: 'November' },
        { value: 12, label: 'December' }
      ],
    };
  },

  computed: {
    years() {
      return [...this.availableYears].sort((a, b) => b - a);
    },
    periodLabels() {
      return {
        day: 'Daily',
        month: 'Monthly', 
        year: 'Yearly'
      };
    },
    months() {
      if (!this.selectedYear || !this.availableMonths.length) {
        return [];
      }
      const monthsForYear = this.availableMonths
        .filter(item => item.year === parseInt(this.selectedYear))
        .map(item => ({
          value: parseInt(item.month),
          label: this.allMonths.find(m => m.value === parseInt(item.month))?.label || `Month ${item.month}`
        }))
        .sort((a, b) => a.value - b.value); 

      return monthsForYear;
    },

    summaryCards() {
      const formatters = {
        distance: (v) => `${this.formatNumber(v / 1000)} KM`,
        time: (v) => this.formatTime(v),
        carbon: (v) => `${this.formatNumber(v)} kg CO₂`
      };
      
      return [
        {
          id: 'distance',
          title: 'Total Distance',
          value: formatters.distance(this.totals.distance),
          icon: require('../assets/map.svg'),
          alt: 'distance',
          color: '#2A67AD'
        },
        {
          id: 'time',
          title: 'Total Moving Time',
          value: formatters.time(this.totals.time),
          icon: require('../assets/clock.svg'),
          alt: 'time',
          color: '#F1CB39'
        },
        {
          id: 'carbon',
          title: 'Total Carbon Saving',
          value: formatters.carbon(this.totals.carbon),
          icon: require('../assets/carbon.svg'),
          alt: 'carbon',
          color: '#00887A'
        }
      ];
    }
  },

  async mounted() {
    await this.fetchAvailablePeriods();
    await this.fetchAllData();
  },

  methods: {
    goBackToDashboard() {
      this.$router.push('/dashboard'); 
    },

    formatNumber(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(value || 0);
    },

    getPeriodLabel(period) {
      const labels = {
        day: 'Daily',
        month: 'Monthly',
        year: 'Yearly'
      };
      return labels[period] || period;
    },

    formatDistance(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format((value || 0) / 1000);
    },

    formatTime(seconds) {
      if (!seconds || seconds < 0) return '00:00:00';
      const h = Math.floor(seconds / 3600);
      const m = Math.floor((seconds % 3600) / 60);
      const s = Math.floor(seconds % 60);
      return [h, m, s].map(u => u.toString().padStart(2, '0')).join(':');
    },

    formatCarbonSaving(value) {
      return new Intl.NumberFormat('id-ID', { 
        minimumFractionDigits: 2, 
        maximumFractionDigits: 2 
      }).format(value || 0);
    },

    getApiParams() {
      const params = { period: this.period };
      
      if (this.period === 'month') {
        params.month = this.selectedMonth;
        params.year = this.selectedYear;
      } else if (this.period === 'year') {
        params.year = this.selectedYear;
      }
      
      console.log('API Params:', params);
      return params;
    },

    async fetchAvailablePeriods() {
      try {
        const response = await axios.get(`${API_BASE_URL}/available-periods`);
        
        if (response.data.status === 200) {
          this.availableMonths = response.data.data.months || [];
          this.availableYears = response.data.data.years || [];
          
          console.log('Available periods:', response.data.data);
          
          if (this.availableYears.length > 0) {
            const currentYear = new Date().getFullYear();
            this.selectedYear = this.availableYears.includes(currentYear) 
              ? currentYear 
              : Math.max(...this.availableYears);
          }
          
          if (this.availableMonths.length > 0) {
            const currentMonth = new Date().getMonth() + 1;
            const monthsForSelectedYear = this.availableMonths
              .filter(item => item.year === parseInt(this.selectedYear))
              .map(item => parseInt(item.month));
            
            if (monthsForSelectedYear.includes(currentMonth)) {
              this.selectedMonth = currentMonth;
            } else if (monthsForSelectedYear.length > 0) {
              this.selectedMonth = Math.max(...monthsForSelectedYear);
            }
          }
        }
      } catch (error) {
        console.error('Error fetching available periods:', error);
        this.selectedYear = new Date().getFullYear();
        this.selectedMonth = new Date().getMonth() + 1;
      }
    },
    async fetchAllData() {
      this.isLoading = true;
      this.error = null;
      
      try {
        const athleteId = parseInt(this.$route.params.id, 10);
        const params = this.getApiParams();
        console.log('Total athlete:', `${API_BASE_URL}/total-athlete`, 'with params:', params);
        console.log('Activities:', `${API_BASE_URL}/athlete/${athleteId}`, 'with params:', params);

        const [athleteDataRes, activitiesRes] = await Promise.all([
          axios.get(`${API_BASE_URL}/total-athlete`, { params }),
          axios.get(`${API_BASE_URL}/athlete/${athleteId}`, { params })
        ]);

        if (athleteDataRes.data.status === 200 && athleteDataRes.data.data) {
          const athleteData = athleteDataRes.data.data.find(item => item.id_athlete === athleteId);
          
          if (athleteData) {
            this.totals = {
              distance: parseFloat(athleteData.total_distance) || 0,
              time: parseInt(athleteData.total_time) || 0,
              carbon: parseFloat(athleteData.total_carbon_saving) || 0,
              activity: parseInt(athleteData.total_activity) || 0
            };
            
            if (!this.athleteName && athleteData.athlete_firstname) {
              this.athleteName = `${athleteData.athlete_firstname} ${athleteData.athlete_lastname}`;
            }
          } else {
            this.totals = { distance: 0, time: 0, carbon: 0, activity: 0 };
            console.log("No data found for athlete ID:", athleteId, "in selected period");
          }
        } else {
          this.totals = { distance: 0, time: 0, carbon: 0, activity: 0 };
        }

        if (activitiesRes.data && activitiesRes.data.data) {
          const athleteActivities = activitiesRes.data.data;
          if (!this.athleteName && athleteActivities.length > 0) {
            this.athleteName = `${athleteActivities[0].athlete_firstname} ${athleteActivities[0].athlete_lastname}`;
          }
          this.activities = athleteActivities;
        } else {
          this.activities = [];
          console.log("No activities found for this athlete in selected period.");
        }

      } catch (error) {
        console.error("❌ Error fetching data:", error);
        if (error.response) {
          console.error("Response error:", error.response.data);
          console.error("Response status:", error.response.status);
        }
        
        if (error.response) {
          if (error.response.status === 400) {
            this.error = "Invalid period selected. Please try again.";
          } else if (error.response.status === 404) {
            this.error = "Athlete not found.";
          } else {
            this.error = "Error fetching data. Please try again later.";
          }
        } else {
          this.error = "Network error. Please check your connection.";
        }

        this.totals = { distance: 0, time: 0, carbon: 0, activity: 0 };
        this.activities = [];
        
      } finally {
        this.isLoading = false;
      }
    },
    async updatePeriod(period) {
      console.log('Updating period to:', period);
      this.period = period;
      if (period === 'month') {
        if (this.availableYears.length > 0 && !this.availableYears.includes(this.selectedYear)) {
          this.selectedYear = Math.max(...this.availableYears);
        }
        const monthsForYear = this.availableMonths
          .filter(item => item.year === parseInt(this.selectedYear))
          .map(item => parseInt(item.month));
        
        if (monthsForYear.length > 0 && !monthsForYear.includes(this.selectedMonth)) {
          this.selectedMonth = Math.max(...monthsForYear);
        }
      } else if (period === 'year') {
        if (this.availableYears.length > 0 && !this.availableYears.includes(this.selectedYear)) {
          this.selectedYear = Math.max(...this.availableYears);
        }
      }
      
      await this.fetchAllData();
    },
    async onMonthChange() {
      console.log('Month changed to:', this.selectedMonth, 'Year:', this.selectedYear);
      await this.fetchAllData();
    },

    async onYearChange() {
      console.log('Year changed to:', this.selectedYear);
      if (this.period === 'month') {
        const monthsForYear = this.availableMonths
          .filter(item => item.year === parseInt(this.selectedYear))
          .map(item => parseInt(item.month));
        
        if (monthsForYear.length > 0 && !monthsForYear.includes(this.selectedMonth)) {
          this.selectedMonth = Math.max(...monthsForYear);
        }
      }
      
      await this.fetchAllData();
    }
  },
};
</script>

<style scoped>
/* Navbar */
.container-banner img {
  width: 180px;
  height: auto;
}

.container-fluid {
  background-color: #002147;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  padding: 1rem 2rem;
}

.navbar-brand {
  margin-left: 4rem; /* Geser text ke kanan biar ga ketutup button */
}

/* Navbar Back Button */
.navbar-back-btn {
  position: absolute;
  left: 2rem;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.3);
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.navbar-back-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.5);
  transform: translateX(-3px);
}

.navbar-back-btn i {
  font-size: 0.875rem;
}

.text-primary {
  color: #002147 !important;
}

/* Section Headers */
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
  gap: 1rem;
}

/* Header Left - contains back button and title */
.header-left {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

/* Back Button Styling */
.back-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  background: linear-gradient(135deg, #2A67AD 0%, #4f91dc 100%);
  color: white;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 500;
  font-size: 0.875rem;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(42, 103, 173, 0.3);
  align-self: flex-start;
}

.back-btn:hover {
  background: linear-gradient(135deg, #1a4a7a 0%, #2A67AD 100%);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(42, 103, 173, 0.4);
}

.back-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 8px rgba(42, 103, 173, 0.3);
}

.back-btn i {
  font-size: 0.875rem;
}

/* Period Container */
.period-container {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  flex-wrap: wrap;
}

/* Period Filter Buttons */
.period-filters {
  display: flex;
  gap: 0.5rem;
  background: #f8f9fa;
  padding: 0.25rem;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.period-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 8px;
  background: transparent;
  color: #6c757d;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  white-space: nowrap;
}

.period-btn:hover {
  background: rgba(42, 103, 173, 0.1);
  color: #2A67AD;
}

.period-btn.active {
  background: #2A67AD;
  color: white;
  box-shadow: 0 2px 8px rgba(42, 103, 173, 0.3);
}

/* Dropdown Styling */
.dropdown-container {
  position: relative;
}

.period-dropdown {
  padding: 0.5rem 1rem;
  border: 2px solid #e9ecef;
  border-radius: 12px;
  background: white;
  color: #495057;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 120px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.period-dropdown:hover {
  border-color: #2A67AD;
  box-shadow: 0 4px 12px rgba(42, 103, 173, 0.2);
}

.period-dropdown:focus {
  outline: none;
  border-color: #2A67AD;
  box-shadow: 0 0 0 3px rgba(42, 103, 173, 0.1);
}

.period-dropdown option {
  padding: 0.5rem;
  background: white;
  color: #495057;
}

/* Modern Cards */
.modern-card {
  background: white;
  border-radius: 20px;
  padding: 1.5rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  display: flex;
  align-items: center;
  gap: 1rem;
  border: 1px solid #f0f0f0;
}

.modern-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

.card-icon-section {
  width: 60px;
  height: 60px;
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.card-icon {
  width: 30px;
  height: 30px;
  filter: brightness(0) invert(1);
}

.card-content {
  flex: 1;
}

.card-title {
  margin: 0 0 0.5rem 0;
  font-size: 0.875rem;
  color: #6c757d;
  font-weight: 500;
}

.card-value {
  margin: 0;
  font-size: 1.5rem;
  font-weight: 700;
  color: #002147;
}

/* Modern Table */
.table-container {
  background: white;
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid #f0f0f0;
  overflow: hidden;
  max-height: 400px;
  overflow-y: auto;
  overflow-x: auto;
}

.modern-table {
  width: 100%;
  border-collapse: collapse;
  margin: 0;
  min-width: 120px;
}

.modern-table thead {
  background: linear-gradient(135deg, #2A67AD 0%, #4f91dc 100%);
  color: white;
  position: sticky;
  top: 0;
  z-index: 10;
}

.modern-table th {
  padding: 1rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.875rem;
  letter-spacing: 0.5px;
  border: none;
}

.modern-table td {
  padding: 1rem;
  border-bottom: 1px solid #f8f9fa;
  font-size: 0.875rem;
  color: #495057;
}

.table-row {
  cursor: pointer;
  transition: all 0.3s ease;
}

.table-row:hover {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  transform: scale(1.01);
}

.table-row:last-child td {
  border-bottom: none;
}

.commuter-name {
  font-weight: 600;
  color: #2A67AD;
}

.carbon-value {
  font-weight: 600;
  color: #00887A;
}

.route-name {
  font-weight: 500;
  color: #495057;
}

/* Loading and Error States */
.modern-alert {
  border-radius: 12px;
  border: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Responsive Design */
@media (max-width: 768px) {
  .section-header {
    flex-direction: column;
    align-items: stretch;
  }

  .header-left {
    align-items: stretch;
  }

  .navbar-back-btn {
    left: 1rem;
  }

  .navbar-brand {
    margin-left: 2rem; /* Reduce margin on tablet */
  }

  .period-filters {
    justify-content: center;
  }

  .modern-card {
    flex-direction: column;
    text-align: center;
  }
  
  .modern-table {
    font-size: 0.75rem;
  }
  
  .modern-table th,
  .modern-table td {
    padding: 0.5rem;
  }
}

@media (max-width: 576px) {
  .period-btn {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
  }
  
  .card-value {
    font-size: 1.2rem;
  }

  .back-btn {
    font-size: 0.8rem;
    padding: 0.4rem 0.8rem;
  }

  .back-btn span {
    display: none; /* Hide text on very small screens, show only icon */
  }

  .navbar-back-btn {
    left: 0.5rem;
    width: 35px;
    height: 35px;
  }

  .navbar-brand {
    margin-left: 1.5rem; /* Even less margin on mobile */
    font-size: 1.2rem; /* Slightly smaller text on mobile */
  }
}

/* Scrollbar Styling */
.table-container::-webkit-scrollbar {
  width: 6px;
}

.table-container::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.table-container::-webkit-scrollbar-thumb {
  background: #2A67AD;
  border-radius: 3px;
}

.table-container::-webkit-scrollbar-thumb:hover {
  background: #1a4a7a;
}
</style>