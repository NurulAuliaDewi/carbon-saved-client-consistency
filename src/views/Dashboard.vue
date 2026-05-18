<template>
  <div>
    <!-- Navbar -->
    <nav class="navbar">
      <div class="container-banner clickable-logo" @click="goToLandingPage">
        <img src="../assets/logob2w.png" alt="Logo">
        <div class="logo-overlay">
          <span>← Back to Landing Page</span>
        </div>
      </div>
      <div class="container-fluid">
        <span class="navbar-brand mb-0 h2 text-white">DASHBOARD</span>
      </div>
    </nav>
    <div class="content m-4">
      <div class="section-header">
        <h4 class="text-primary fw-bold mb-4">Bike Commuting Data</h4>
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
      <div class="charts-grid">
        <div v-for="chart in chartInfo" :key="chart.id" class="chart-wrapper">
          <div class="chart-card">
            <div class="chart-body">
              <div :id="chart.id" class="chart-container"></div>
            </div>
          </div>
        </div>
      </div>
      <div class="section-header">
        <h4 class="text-primary fw-bold mb-4">Summary Commuting</h4>
        <div class="controls-section">
          <div class="search-container">
            <div class="search-input-wrapper">
              <i class="fas fa-search search-icon"></i>
              <input 
                v-model="searchQuery" 
                placeholder="Search by Username"
                class="search-input"
              />
            </div>
          </div>
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
      </div>
      <div class="table-container">
        <table class="modern-table">
          <thead>
            <tr>
              <th>No</th>
              <th>Commuter</th>
              <th>Distance (KM)</th>
              <th>Moving Time</th>
              <th>Carbon Saving (kg CO<sub>2</sub>)</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="(athlete, index) in filteredAthletes" 
              :key="athlete.id_athlete" 
              @click="$router.push(`/detail/${athlete.id_athlete}`)"
              class="table-row"
            >
              <td>{{ index + 1 }}</td>
              <td class="commuter-name">{{ athlete.fullName }}</td>
              <td>{{ athlete.distance }}</td>
              <td>{{ athlete.movingTime }}</td>
              <td class="carbon-value">{{ athlete.carbonSaving }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-if="isLoading" class="loading-container">
        <div class="loading-spinner"></div>
        <p>Loading data...</p>
      </div>
      <div v-if="!isLoading && athletes.length === 0" class="no-data-container">
        <p>No data available for the selected period.</p>
      </div>
    </div>
  </div>
</template>

<script>
import ApexCharts from 'apexcharts';
import axios from 'axios';

// const API_BASE_URL = 'https://carbonsaved.bike2work.id/api';
const API_BASE_URL = 'http://localhost:5001/api';
const THRESHOLDS = {
  day: { short: 15, mid: 30 },
  month: { short: 300, mid: 500 },
  year: { short: 3600, mid: 6000 }
};

export default {
  name: 'HomeView',
  
  data() {
    return {
      athletes: [],
      totals: { distance: 0, time: 0, carbon: 0 },
      period: 'day',
      selectedMonth: new Date().getMonth() + 1, 
      selectedYear: new Date().getFullYear(), 
      searchQuery: '',
      chartInstances: [],
      isLoading: false,
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
      chartInfo: [
        { id: 'chart1', type: 'line' },
        { id: 'chart2', type: 'bar' },
        { id: 'chart3', type: 'donut' },
        { id: 'chart4', type: 'area' },
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
        { id: 'distance', title: 'Total Distance', value: formatters.distance(this.totals.distance), 
          icon: require('../assets/map.svg'), alt: 'distance', color: '#2A67AD' },
        { id: 'time', title: 'Total Moving Time', value: formatters.time(this.totals.time), 
          icon: require('../assets/clock.svg'), alt: 'time', color: '#F1CB39' },
        { id: 'carbon', title: 'Total Carbon Saving', value: formatters.carbon(this.totals.carbon), 
          icon: require('../assets/carbon.svg'), alt: 'carbon', color: '#00887A' }
      ];
    },

    filteredAthletes() {
      if (!this.searchQuery.trim()) return this.athletes;
      const query = this.searchQuery.toLowerCase();
      return this.athletes.filter(a => a.fullName.toLowerCase().includes(query));
    }
  },

  async mounted() {
    await this.fetchAvailablePeriods();
    await this.fetchAllData();
  },

  beforeDestroy() {
    this.chartInstances.forEach(chart => {
      try {
        chart.destroy();
      } catch (error) {
        console.warn("Error destroying chart:", error);
      }
    });
  },

  methods: {
    // Utility functions
    goToLandingPage() {
      this.$router.push('/');
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
    formatTime(seconds) {
      if (!seconds || seconds < 0) return '00:00:00';
      const h = Math.floor(seconds / 3600);
      const m = Math.floor((seconds % 3600) / 60);
      const s = Math.floor(seconds % 60);
      return [h, m, s].map(u => u.toString().padStart(2, '0')).join(':');
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
      try {
        const params = this.getApiParams();
        console.log('Fetching data with params:', params);
        const [totalRes, athleteRes] = await Promise.all([
          axios.get(`${API_BASE_URL}/total`, { params }),
          axios.get(`${API_BASE_URL}/total-athlete`, { params })
        ]);

        console.log('Total Response:', totalRes.data);
        console.log('Athlete Response:', athleteRes.data);

        if (totalRes.data.status === 200 && totalRes.data.data && totalRes.data.data.length > 0) {
          const data = totalRes.data.data[0];
          this.totals = {
            distance: data.total_distance || 0,
            time: data.total_time || 0,
            carbon: data.total_carbon_saving || 0
          };
        } else {
          this.totals = { distance: 0, time: 0, carbon: 0 };
        }

        if (athleteRes.data.status === 200 && athleteRes.data.data) {
          this.athletes = athleteRes.data.data.map(item => ({
            fullName: `${item.athlete_firstname || ''} ${item.athlete_lastname || ''}`.trim(),
            distance: this.formatNumber((item.total_distance || 0) / 1000),
            carbonSaving: parseFloat(item.total_carbon_saving || 0).toFixed(2),
            movingTime: this.formatTime(item.total_time || 0),
            id_athlete: item.id_athlete
          }));

          this.updateCharts(athleteRes.data.data);
        } else {
          this.athletes = [];
          this.updateCharts([]);
        }

      } catch (error) {
        console.error("Error fetching data:", error);
        if (error.response) {
          console.error("Response error:", error.response.data);
          if (error.response.status === 400) {
            alert("Invalid period selected. Please try again.");
          }
        }
        this.totals = { distance: 0, time: 0, carbon: 0 };
        this.athletes = [];
        this.updateCharts([]);
        
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
    },

    updateCharts(apiData) {
      this.chartInstances.forEach(chart => {
        try {
          chart.destroy();
        } catch (error) {
          console.warn("Error destroying chart:", error);
        }
      });
      this.chartInstances = [];

      if (!apiData || !apiData.length) {
        console.log("No data available for charts");
        return;
      }

      const chartData = this.prepareChartData(apiData);
      const options = this.getChartOptions(chartData);

      this.$nextTick(() => {
        setTimeout(() => {
          this.chartInfo.forEach(chart => {
            const el = document.querySelector(`#${chart.id}`);
            if (el && el.classList) {
              try {
                const instance = new ApexCharts(el, options[chart.type]);
                instance.render();
                this.chartInstances.push(instance);
              } catch (error) {
                console.error(`Error rendering chart ${chart.id}:`, error);
              }
            } else {
              console.warn(`Chart element #${chart.id} not found or not ready`);
            }
          });
        }, 100);
      });
    },

    prepareChartData(apiData) {
      const totalActivity = apiData.reduce((sum, item) => sum + parseFloat(item.total_activity || 0), 0);
      const top10Distance = [...apiData].sort((a, b) => (b.total_distance || 0) - (a.total_distance || 0)).slice(0, 10);
      const top10Carbon = [...apiData].sort((a, b) => (b.total_carbon_saving || 0) - (a.total_carbon_saving || 0)).slice(0, 10);
      
      const distances = apiData.map(item => (item.total_distance || 0) / 1000);
      const { short, mid } = THRESHOLDS[this.period];
      const groups = { short: 0, mid: 0, long: 0 };
      
      distances.forEach(d => {
        if (d <= short) groups.short++;
        else if (d <= mid) groups.mid++;
        else groups.long++;
      });

      return {
        totalActivity,
        distanceGroups: groups,
        top10Distance: {
          names: top10Distance.map(item => `${item.athlete_firstname || ''} ${item.athlete_lastname || ''}`.trim()),
          values: top10Distance.map(item => ((item.total_distance || 0) / 1000).toFixed(2))
        },
        top10Carbon: {
          names: top10Carbon.map(item => `${item.athlete_firstname || ''} ${item.athlete_lastname || ''}`.trim()),
          values: top10Carbon.map(item => parseFloat(item.total_carbon_saving || 0).toFixed(2))
        }
      };
    },

    getChartOptions(data) {
      let periodLabel = this.period.charAt(0).toUpperCase() + this.period.slice(1);
      
      // Add specific month/year to chart titles
      if (this.period === 'month') {
        const monthName = this.allMonths.find(m => m.value === this.selectedMonth)?.label || '';
        periodLabel = `${monthName} ${this.selectedYear}`;
      } else if (this.period === 'year') {
        periodLabel = this.selectedYear.toString();
      } else if (this.period === 'day') {
        periodLabel = 'Today';
      }
      
      const { short, mid } = THRESHOLDS[this.period];
      
      // Base chart configuration untuk konsistensi
      const baseConfig = {
        chart: {
          toolbar: {
            show: false
          },
          background: 'transparent',
          fontFamily: 'inherit',
          animations: {
            enabled: true,
            easing: 'easeinout',
            speed: 800
          }
        },
        colors: ['#2A67AD', '#F1CB39', '#00887A', '#dc3545', '#fd7e14'],
        dataLabels: {
          enabled: false
        },
        grid: {
          borderColor: '#f1f1f1',
          strokeDashArray: 4
        },
        legend: {
          position: 'bottom',
          horizontalAlign: 'center',
          fontSize: '12px',
          fontWeight: 500,
          markers: {
            width: 12,
            height: 12,
            radius: 6
          }
        }
      };
      
      return {
        line: {
          ...baseConfig,
          chart: {
            ...baseConfig.chart,
            type: 'line',
            height: 350
          },
          series: [{ name: 'Distance (KM)', data: data.top10Distance.values }],
          xaxis: { 
            categories: data.top10Distance.names,
            labels: {
              rotate: -45,
              style: {
                fontSize: '11px'
              }
            }
          },
          yaxis: {
            title: {
              text: 'Distance (KM)',
              style: {
                fontSize: '12px',
                fontWeight: 600
              }
            }
          },
          stroke: {
            width: 3,
            curve: 'smooth'
          },
          markers: {
            size: 6,
            strokeWidth: 2,
            fillOpacity: 1
          },
          title: { 
            text: `Top 10 Distance - ${periodLabel}`, 
            align: 'center',
            style: {
              fontSize: '16px',
              fontWeight: 600,
              color: '#002147'
            }
          }
        },
        bar: {
          ...baseConfig,
          chart: {
            ...baseConfig.chart,
            type: 'bar',
            height: 350
          },
          series: [{ name: 'Total Activity', data: [data.totalActivity] }],
          xaxis: { 
            categories: ['Total Activity'],
            labels: {
              style: {
                fontSize: '12px',
                fontWeight: 600
              }
            }
          },
          yaxis: {
            title: {
              text: 'Number of Activities',
              style: {
                fontSize: '12px',
                fontWeight: 600
              }
            }
          },
          plotOptions: {
            bar: {
              borderRadius: 8,
              columnWidth: '60%'
            }
          },
          title: { 
            text: `Total Activity - ${periodLabel}`, 
            align: 'center',
            style: {
              fontSize: '16px',
              fontWeight: 600,
              color: '#002147'
            }
          }
        },
        donut: {
          ...baseConfig,
          chart: {
            ...baseConfig.chart,
            type: 'donut',
            height: 350
          },
          series: [data.distanceGroups.short, data.distanceGroups.mid, data.distanceGroups.long],
          labels: [`Short (≤${short}km)`, `Mid (≤${mid}km)`, `Long (>${mid}km)`],
          plotOptions: {
            pie: {
              donut: {
                size: '65%',
                labels: {
                  show: true,
                  total: {
                    show: true,
                    label: 'Total Commuters',
                    fontSize: '14px',
                    fontWeight: 600,
                    color: '#002147'
                  }
                }
              }
            }
          },
          title: { 
            text: `Distance Range - ${periodLabel}`, 
            align: 'center',
            style: {
              fontSize: '16px',
              fontWeight: 600,
              color: '#002147'
            }
          }
        },
        area: {
          ...baseConfig,
          chart: {
            ...baseConfig.chart,
            type: 'area',
            height: 350
          },
          series: [{ name: 'Carbon Saving (kg CO₂)', data: data.top10Carbon.values }],
          xaxis: { 
            categories: data.top10Carbon.names,
            labels: {
              rotate: -45,
              style: {
                fontSize: '11px'
              }
            }
          },
          yaxis: {
            title: {
              text: 'Carbon Saving (kg CO₂)',
              style: {
                fontSize: '12px',
                fontWeight: 600
              }
            }
          },
          stroke: {
            width: 2,
            curve: 'smooth'
          },
          fill: {
            type: 'gradient',
            gradient: {
              shadeIntensity: 1,
              opacityFrom: 0.7,
              opacityTo: 0.3,
              stops: [0, 90, 100]
            }
          },
          title: { 
            text: `Top 10 Carbon Saved - ${periodLabel}`, 
            align: 'center',
            style: {
              fontSize: '16px',
              fontWeight: 600,
              color: '#002147'
            }
          }
        }
      };
    }
  }
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

/* Charts Grid */
.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 1.5rem;
  margin-bottom: 3rem;
}

.chart-wrapper {
  display: flex;
  flex-direction: column;
}

/* Chart Cards */
.chart-card {
  background: white;
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid #f0f0f0;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chart-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 25px rgba(0, 0, 0, 0.1);
}

.chart-body {
  padding: 1.5rem;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.chart-container {
  height: 350px;
  width: 100%;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Controls Section */
.controls-section {
  display: flex;
  gap: 1rem;
  align-items: center;
  flex-wrap: wrap;
}

.search-container {
  flex: 1;
  min-width: 200px;
}

.search-input-wrapper {
  position: relative;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  border: 1px solid #e9ecef;
  overflow: hidden;
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: #6c757d;
  z-index: 1;
}

.search-input {
  width: 100%;
  padding: 0.75rem 1rem 0.75rem 2.5rem;
  border: none;
  outline: none;
  background: transparent;
  color: #495057;
  font-size: 0.875rem;
}

.search-input::placeholder {
  color: #6c757d;
}

.search-input:focus {
  box-shadow: 0 0 0 3px rgba(42, 103, 173, 0.1);
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

/* Responsive Design */
@media (max-width: 768px) {
  .section-header {
    flex-direction: column;
    align-items: stretch;
  }

  .period-filters {
    justify-content: center;
  }

  .controls-section {
    flex-direction: column;
    align-items: stretch;
  }

  .modern-card {
    flex-direction: column;
    text-align: center;
  }

  .charts-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .chart-container {
    height: 300px;
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

  .charts-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .chart-container {
    height: 280px;
  }
}

@media (max-width: 1200px) {
  .charts-grid {
    grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
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

.clickable-logo {
  position: relative;
  cursor: pointer;
  transition: all 0.3s ease;
  overflow: hidden;
}

.clickable-logo:hover {
  transform: scale(1.05);
}

.logo-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(42, 103, 173, 0.9);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.875rem;
  font-weight: 500;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.clickable-logo:hover .logo-overlay {
  opacity: 1;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .nav-buttons {
    flex-direction: column;
    gap: 0.25rem;
  }
  
  .nav-btn {
    padding: 0.4rem 0.8rem;
    font-size: 0.75rem;
  }
  
  .container-fluid {
    flex-direction: column;
    gap: 0.5rem;
    text-align: center;
  }
  
  .fab-container {
    bottom: 1rem;
    left: 1rem;
  }
  
  .fab {
    width: 48px;
    height: 48px;
    font-size: 1rem;
  }
}

@media (max-width: 576px) {
  .breadcrumb-container {
    padding: 0.5rem;
  }
  
  .breadcrumb {
    font-size: 0.75rem;
  }
}
</style>