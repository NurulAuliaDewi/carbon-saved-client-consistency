<template>
  <div class="landing">
    <div class="hero-content">

      <div class="logo-container">
        <div class="logo">
          <i class="fas fa-bicycle"></i>
        </div>
      </div>

      <h1 class="main-title">
        <span class="gradient-text">Bike to Work</span>
        <span class="title-sub">Consistency Tracker</span>
      </h1>

      <p class="desc">
        Track your monthly commuting streak and see how you stack up against fellow riders.
      </p>

      <div class="summary-section">

        <div v-if="loadingMonthlyFreq" class="loading-container">
          <div class="loading-spinner-large">⟳</div>
          <p>Loading...</p>
        </div>

        <div v-else class="main-grid">

          <!-- LEFT COL -->
          <div class="left-col">

            <!-- Filter -->
            <div class="panel filter-panel">
              <div class="panel-header">
                <span class="panel-icon"><i class="fas fa-sliders-h"></i></span>
                <h3 class="panel-title">Period</h3>
              </div>

              <!-- Mode toggle -->
              <div class="mode-toggle">
                <button
                  class="mode-btn"
                  :class="{ active: filterMode === 'month' }"
                  @click="switchFilterMode('month')"
                >Monthly</button>
                <button
                  class="mode-btn"
                  :class="{ active: filterMode === 'range' }"
                  @click="switchFilterMode('range')"
                >Custom Range</button>
              </div>

              <!-- Monthly mode -->
              <div v-if="filterMode === 'month'" class="filter-stack">
                <div class="filter-group">
                  <label class="filter-label">Year</label>
                  <select v-model="monthlyFilterYear" @change="onMonthlyYearChange" class="dropdown">
                    <option v-for="y in availableYears" :key="y" :value="y">{{ y }}</option>
                  </select>
                </div>
                <div class="filter-group">
                  <label class="filter-label">Month</label>
                  <select v-model="monthlyFilterMonth" @change="onMonthlyFilterChange" class="dropdown">
                    <option v-for="m in filteredMonthlyMonths" :key="m.value" :value="parseInt(m.value)">
                      {{ m.monthLabel }}
                    </option>
                  </select>
                </div>
              </div>

              <!-- Custom range mode -->
              <div v-if="filterMode === 'range'" class="filter-stack">
                <div class="filter-group">
                  <label class="filter-label">Start Date</label>
                  <input type="date" v-model="startDate" class="dropdown date-input" />
                </div>
                <div class="filter-group">
                  <label class="filter-label">End Date</label>
                  <input type="date" v-model="endDate" class="dropdown date-input" />
                </div>
                <button class="apply-btn" @click="applyDateRange">
                  <i class="fas fa-check"></i> Apply
                </button>
              </div>
            </div>

            <!-- Total Participant Card -->
            <div class="panel stat-card">
              <div class="stat-icon"><i class="fas fa-users"></i></div>
              <div class="stat-value">{{ totalParticipants }}</div>
              <div class="stat-label">Total Participants</div>
              <div class="stat-sub">Active commuters this period</div>
            </div>

            <!-- Frequency Chart -->
            <div class="panel freq-panel">
              <div class="panel-header">
                <span class="panel-icon"><i class="fas fa-chart-bar"></i></span>
                <h3 class="panel-title">Activity Frequency</h3>
              </div>
              <div class="freq-chart">
                <div class="chart-row" v-for="item in monthlyFreqDistribution" :key="item.label">
                  <div class="chart-label">{{ item.label }}</div>
                  <div class="chart-track">
                    <div class="chart-fill" :style="`width: ${item.percentage}%`"></div>
                  </div>
                  <div class="chart-count">{{ item.count }}</div>
                </div>
              </div>
            </div>

          </div>

          <!-- RIGHT: Leaderboard -->
          <div class="panel right-panel">
            <div class="panel-header">
              <span class="panel-icon"><i class="fas fa-trophy"></i></span>
              <h3 class="panel-title">Commuter Leaderboard</h3>
              <span class="panel-sub">Ranked by active days</span>
            </div>

            <div class="search-wrap">
              <i class="fas fa-search search-icon"></i>
              <input
                v-model="athleteSearch"
                type="text"
                placeholder="Search commuter..."
                class="search-input"
                @input="onAthleteSearch"
              />
            </div>

            <p class="table-count" v-if="athleteSearch">
              {{ filteredMonthlyAthletes.length }} of {{ monthlyAthletes.length }} shown
            </p>

            <div v-if="filteredMonthlyAthletes.length === 0" class="no-data">
              <i class="fas fa-info-circle"></i>
              <p>No data available for this period</p>
            </div>

            <div v-else class="athlete-table-wrap">
              <table class="athlete-table">
                <thead>
                  <tr>
                    <th class="col-no">#</th>
                    <th class="col-name">Name</th>
                    <th class="col-center">Days</th>
                    <th class="col-center">Progress</th>
                    <th class="col-center">Distance</th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="athlete in filteredMonthlyAthletes"
                    :key="athlete.id"
                    :class="{
                      'row-gold':   getRank(athlete) === 0,
                      'row-silver': getRank(athlete) === 1,
                      'row-bronze': getRank(athlete) === 2
                    }"
                  >
                    <td class="col-no">
                      <span v-if="getRank(athlete) === 0">👑</span>
                      <span v-else-if="getRank(athlete) === 1">🥈</span>
                      <span v-else-if="getRank(athlete) === 2">🥉</span>
                      <span v-else class="rank-number">#{{ getRank(athlete) + 1 }}</span>
                    </td>
                    <td class="col-name">{{ athlete.name }}</td>
                    <td class="col-center">
                      <span class="days-badge">{{ athlete.active_days }}d</span>
                    </td>
                    <td class="col-center">
                      <div class="bar-mini-track">
                        <div class="bar-mini-fill" :style="`width: ${getActiveDayPercent(athlete.active_days)}%`"></div>
                      </div>
                    </td>
                    <td class="col-center km-txt">{{ formatDistance(athlete.total_distance) }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'LandingPage',
  data() {
    return {
      filterMode: 'month',
      startDate: '',
      endDate: '',
      loadingMonthlyFreq: false,
      monthlyFilterMonth: new Date().getMonth() + 1,
      monthlyFilterYear: new Date().getFullYear(),
      availableMonths: [],
      availableYears: [],
      filteredMonthlyMonths: [],
      athleteSearch: '',
      filteredMonthlyAthletes: [],
      totalParticipants: '-',
      monthlyFreqDistribution: [
        { label: '1–4 days',   percentage: 0, count: '-', key: '1_to_4_days'   },
        { label: '5–9 days',   percentage: 0, count: '-', key: '5_to_9_days'   },
        { label: '10–14 days', percentage: 0, count: '-', key: '10_to_14_days' },
        { label: '15–19 days', percentage: 0, count: '-', key: '15_to_19_days' },
        { label: '20+ days',   percentage: 0, count: '-', key: '20_plus_days'  }
      ],
      monthlyAthletes: [],
      apiBaseUrl: process.env.VUE_APP_API_URL || 'http://localhost:5001/api'
      //|| 'https://consistency.bike2work.id/api'
    }
  },

  async mounted() {
    await this.fetchAvailableMonths()
    await this.fetchMonthlyActivityFrequency()
  },

  methods: {
    formatNumber(num) {
      if (!num || isNaN(num)) return '-'
      const n = typeof num === 'string' ? parseFloat(num) : num
      return new Intl.NumberFormat('id-ID', { minimumFractionDigits: 0, maximumFractionDigits: 0 }).format(Math.round(n))
    },
    formatNumberWithDecimals(num, decimals = 1) {
      if (!num || isNaN(num)) return '-'
      const n = typeof num === 'string' ? parseFloat(num) : num
      return new Intl.NumberFormat('id-ID', { minimumFractionDigits: decimals, maximumFractionDigits: decimals }).format(n)
    },
    formatDistance(distance) {
      if (!distance) return '0 km'
      const km = (typeof distance === 'string' ? parseFloat(distance) : distance) / 1000
      return km >= 1000
        ? `${this.formatNumber(km)} km`
        : `${this.formatNumberWithDecimals(km, 1)} km`
    },
    getActiveDayPercent(activeDays) {
      if (!this.monthlyAthletes.length) return 0
      const max = this.monthlyAthletes[0].active_days
      return max > 0 ? Math.round((activeDays / max) * 100) : 0
    },
    getRank(athlete) {
      return this.monthlyAthletes.findIndex(a => a.id === athlete.id)
    },
    onAthleteSearch() {
      const q = this.athleteSearch.toLowerCase().trim()
      this.filteredMonthlyAthletes = q
        ? this.monthlyAthletes.filter(a => a.name.toLowerCase().includes(q))
        : [...this.monthlyAthletes]
    },
    toMonthLabel(year, monthVal) {
      return new Date(year, parseInt(monthVal) - 1, 1).toLocaleString('en-US', { month: 'long' })
    },
    async fetchAvailableMonths() {
      try {
        const response = await axios.get(`${this.apiBaseUrl}/available-periods`)
        if (response.data?.status === 200 && response.data.data) {
          this.availableMonths = response.data.data.months.map(item => ({
            value: item.month.toString(),
            label: new Date(item.year, item.month - 1, 1).toLocaleString('en-US', { month: 'long', year: 'numeric' }),
            year: item.year
          }))
          this.availableYears = response.data.data.years || []
          const now = new Date()
          const currYear = now.getFullYear()
          const currMonth = (now.getMonth() + 1).toString()
          this.monthlyFilterYear = this.availableYears.includes(currYear) ? currYear : (this.availableYears[0] || currYear)
          this.filteredMonthlyMonths = this.availableMonths
            .filter(m => m.year === this.monthlyFilterYear)
            .map(m => ({ ...m, monthLabel: this.toMonthLabel(m.year, m.value) }))
          const found = this.filteredMonthlyMonths.find(m => parseInt(m.value) === parseInt(currMonth))
          this.monthlyFilterMonth = found
            ? parseInt(currMonth)
            : (this.filteredMonthlyMonths[0] ? parseInt(this.filteredMonthlyMonths[0].value) : parseInt(currMonth))
        }
      } catch (error) {
        console.error('Error fetching available months:', error)
        const current = new Date().getFullYear()
        this.availableYears = []
        for (let y = current; y >= 2024; y--) this.availableYears.push(y)
      }
    },
    async fetchMonthlyActivityFrequency() {
      try {
        this.loadingMonthlyFreq = true

        let params = {}
        if (this.filterMode === 'range') {
          params = { start_date: this.startDate, end_date: this.endDate }
        } else {
          params = { month: this.monthlyFilterMonth, year: this.monthlyFilterYear }
        }

        const response = await axios.get(`${this.apiBaseUrl}/monthly-activity-frequency`, { params })
        if (response.data?.status === 200) {
          const { athletes, frequency_distribution } = response.data.data

          this.monthlyAthletes = athletes.map(a => ({
            id:             a.id_athlete,
            name:           [a.athlete_firstname, a.athlete_lastname].filter(Boolean).join(' ') || 'Anonymous',
            active_days:    parseInt(a.active_days) || 0,
            total_distance: a.total_distance
          }))

          this.totalParticipants = this.formatNumber(this.monthlyAthletes.length)

          this.athleteSearch = ''
          this.filteredMonthlyAthletes = [...this.monthlyAthletes]

          const dist = frequency_distribution
          const total = Object.values(dist).reduce((sum, v) => sum + parseInt(v || 0), 0)
          this.monthlyFreqDistribution.forEach(item => {
            const count = parseInt(dist[item.key] || 0)
            item.count = this.formatNumber(count)
            item.percentage = total > 0 ? Math.max(Math.round((count / total) * 100), count > 0 ? 5 : 0) : 0
          })
        }
      } catch (err) {
        console.error('Error fetching monthly activity frequency:', err)
      } finally {
        this.loadingMonthlyFreq = false
      }
    },
    switchFilterMode(mode) {
      this.filterMode = mode
      if (mode === 'range' && !this.startDate) {
        // default: first day of current month → today
        const now = new Date()
        const y = now.getFullYear()
        const m = String(now.getMonth() + 1).padStart(2, '0')
        const d = String(now.getDate()).padStart(2, '0')
        this.startDate = `${y}-${m}-01`
        this.endDate   = `${y}-${m}-${d}`
      }
      if (mode === 'month') {
        this.fetchMonthlyActivityFrequency()
      }
    },
    applyDateRange() {
      if (!this.startDate || !this.endDate) return
      if (this.startDate > this.endDate) {
        alert('Start date must be before end date')
        return
      }
      this.fetchMonthlyActivityFrequency()
    },
    onMonthlyYearChange() {
      this.filteredMonthlyMonths = this.availableMonths
        .filter(m => m.year === this.monthlyFilterYear)
        .map(m => ({ ...m, monthLabel: this.toMonthLabel(m.year, m.value) }))
      if (this.filteredMonthlyMonths.length > 0) {
        this.monthlyFilterMonth = parseInt(this.filteredMonthlyMonths[0].value)
      }
      this.fetchMonthlyActivityFrequency()
    },
    async onMonthlyFilterChange() {
      await this.fetchMonthlyActivityFrequency()
    }
  }
}
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }

.landing {
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #0f1724;
  color: #e8edf5;
  padding: 3rem 2rem 5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.hero-content {
  width: 100%;
  max-width: 1200px;
  animation: fadeUp 0.6s ease-out both;
}
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(18px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ── LOGO ── */
.logo-container { text-align: center; margin-bottom: 1.75rem; }
.logo {
  width: 76px; height: 76px;
  background: #c8f035;
  border-radius: 18px;
  display: inline-flex; align-items: center; justify-content: center;
}
.logo i { font-size: 2rem; color: #0f1724; }

/* ── TITLE ── */
.main-title { text-align: center; margin-bottom: 0.6rem; }
.gradient-text {
  display: block;
  font-size: 2.8rem; font-weight: 800; letter-spacing: -0.5px;
  background: linear-gradient(110deg, #c8f035 20%, #f0e040 80%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
}
.title-sub {
  display: block;
  font-size: 1rem; font-weight: 500;
  color: #6b7e9a;
  letter-spacing: 0.8px;
  text-transform: uppercase;
  margin-top: 0.3rem;
}
.desc {
  text-align: center; max-width: 480px; margin: 0.75rem auto 2.5rem;
  font-size: 0.97rem; line-height: 1.65; color: #7a90a8;
}

/* ── MAIN GRID ── */
.main-grid {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 1.25rem;
  align-items: start;
}

/* ── LEFT COL ── */
.left-col {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  position: sticky;
  top: 2rem;
}

/* ── PANEL ── */
.panel {
  background: #182030;
  border: 1px solid #243047;
  border-radius: 18px;
  padding: 1.5rem;
}
.panel-header {
  display: flex; align-items: center; gap: 0.55rem;
  margin-bottom: 1.25rem;
}
.panel-icon { font-size: 1rem; color: #c8f035; }
.panel-title { font-size: 0.97rem; font-weight: 700; color: #e8edf5; }
.panel-sub { font-size: 0.75rem; color: #3d5470; margin-left: auto; }

/* ── STAT CARD ── */
.stat-card {
  text-align: center;
  padding: 1.75rem 1.5rem;
  border-top: 3px solid #c8f035;
}
.stat-icon {
  font-size: 1.6rem;
  color: #c8f035;
  margin-bottom: 0.75rem;
  opacity: 0.85;
}
.stat-value {
  font-size: 2.6rem;
  font-weight: 800;
  color: #c8f035;
  letter-spacing: -1px;
  line-height: 1;
  margin-bottom: 0.5rem;
}
.stat-label {
  font-size: 0.92rem;
  font-weight: 700;
  color: #e8edf5;
  margin-bottom: 0.3rem;
}
.stat-sub {
  font-size: 0.75rem;
  color: #3d5470;
}

/* ── MODE TOGGLE ── */
.mode-toggle {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1.1rem;
  background: #0f1724;
  border-radius: 10px;
  padding: 4px;
}
.mode-btn {
  flex: 1;
  padding: 0.45rem 0.5rem;
  background: transparent;
  border: none;
  border-radius: 7px;
  color: #3d5470;
  font-size: 0.78rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}
.mode-btn.active {
  background: #c8f035;
  color: #0f1724;
}
.mode-btn:not(.active):hover { color: #7a90a8; }

/* ── DATE INPUT ── */
.date-input {
  color-scheme: dark;
}
.date-input::-webkit-calendar-picker-indicator {
  filter: invert(0.6);
  cursor: pointer;
}

/* ── APPLY BUTTON ── */
.apply-btn {
  width: 100%;
  padding: 0.6rem;
  background: #c8f035;
  border: none;
  border-radius: 9px;
  color: #0f1724;
  font-size: 0.88rem;
  font-weight: 700;
  cursor: pointer;
  transition: opacity 0.2s;
  margin-top: 0.25rem;
}
.apply-btn:hover { opacity: 0.85; }
.apply-btn i { margin-right: 4px; }

/* ── FILTER ── */
.filter-stack { display: flex; flex-direction: column; gap: 1rem; }
.filter-group { display: flex; flex-direction: column; gap: 0.35rem; }
.filter-label {
  font-size: 0.7rem; font-weight: 700;
  color: #c8f035;
  text-transform: uppercase; letter-spacing: 1px;
}
.dropdown {
  width: 100%;
  padding: 0.55rem 0.9rem;
  background: #0f1724;
  border: 1.5px solid #2d3d55;
  border-radius: 9px;
  color: #e8edf5;
  font-size: 0.92rem; font-weight: 500;
  cursor: pointer;
  transition: border-color 0.2s;
}
.dropdown:focus { outline: none; border-color: #c8f035; }
.dropdown option { background: #182030; color: #e8edf5; }

/* ── FREQ CHART ── */
.freq-chart { display: flex; flex-direction: column; gap: 0.9rem; }
.chart-row { display: flex; align-items: center; gap: 0.65rem; }
.chart-label { font-size: 0.78rem; color: #7a90a8; width: 70px; flex-shrink: 0; }
.chart-track {
  flex: 1; height: 8px;
  background: #1e2d42;
  border-radius: 4px; overflow: hidden;
}
.chart-fill {
  height: 100%;
  background: linear-gradient(90deg, #c8f035, #f0e040);
  border-radius: 4px;
  transition: width 0.7s ease;
}
.chart-count { font-size: 0.78rem; font-weight: 700; color: #c8f035; width: 20px; text-align: right; }

/* ── SEARCH ── */
.search-wrap { position: relative; margin-bottom: 0.9rem; }
.search-icon {
  position: absolute; left: 0.85rem; top: 50%; transform: translateY(-50%);
  color: #3d5470; font-size: 0.82rem; pointer-events: none;
}
.search-input {
  width: 100%; padding: 0.55rem 1rem 0.55rem 2.35rem;
  background: #0f1724;
  border: 1.5px solid #2d3d55;
  border-radius: 9px;
  color: #e8edf5; font-size: 0.88rem;
  transition: border-color 0.2s;
}
.search-input:focus { outline: none; border-color: #c8f035; }
.search-input::placeholder { color: #3d5470; }
.table-count { font-size: 0.78rem; color: #3d5470; margin-bottom: 0.7rem; }

/* ── TABLE ── */
.right-panel {
  display: flex;
  flex-direction: column;
}

.athlete-table-wrap {
  max-height: 600px;
  overflow-y: auto; overflow-x: auto;
  border-radius: 12px;
  border: 1px solid #243047;
  scrollbar-width: thin;
  scrollbar-color: #2d3d55 #0f1724;
}
.athlete-table-wrap::-webkit-scrollbar { width: 5px; height: 5px; }
.athlete-table-wrap::-webkit-scrollbar-track { background: #0f1724; }
.athlete-table-wrap::-webkit-scrollbar-thumb { background: #2d3d55; border-radius: 3px; }
.athlete-table { width: 100%; border-collapse: collapse; font-size: 0.87rem; }
.athlete-table thead tr { position: sticky; top: 0; z-index: 2; }
.athlete-table th {
  padding: 10px 14px;
  color: #3d5470; font-weight: 700; font-size: 0.72rem;
  text-transform: uppercase; letter-spacing: 0.5px;
  border-bottom: 1px solid #243047;
  background: #141e2e;
  white-space: nowrap;
}
.athlete-table td {
  padding: 11px 14px;
  color: #c8d6e8;
  border-bottom: 1px solid #1e2d42;
  vertical-align: middle;
}
.athlete-table tbody tr:last-child td { border-bottom: none; }
.athlete-table tbody tr:hover { background: #1e2d42; transition: background 0.15s; }

.row-gold   { background: rgba(200, 240, 53, 0.06) !important; }
.row-silver { background: rgba(180, 190, 210, 0.05) !important; }
.row-bronze { background: rgba(205, 127, 50, 0.05) !important; }

.col-no     { text-align: center; width: 50px; font-size: 1rem; }
.col-name   { font-weight: 600; color: #e8edf5; }
.col-center { text-align: center; }
.rank-number { font-size: 0.8rem; font-weight: 700; color: #c8f035; }
.days-badge {
  display: inline-block;
  background: rgba(200, 240, 53, 0.12);
  color: #c8f035;
  border-radius: 20px; padding: 2px 11px;
  font-weight: 700; font-size: 0.8rem;
}
.bar-mini-track {
  width: 70px; height: 6px;
  background: #1e2d42;
  border-radius: 3px; overflow: hidden;
  display: inline-block; vertical-align: middle;
}
.bar-mini-fill {
  height: 100%; border-radius: 3px;
  background: linear-gradient(90deg, #c8f035, #f0e040);
  transition: width 0.5s ease;
}
.km-txt { color: #c8f035; font-weight: 600; }

/* ── LOADING ── */
.loading-container {
  display: flex; flex-direction: column; align-items: center;
  justify-content: center; padding: 4rem; color: #6b7e9a; gap: 1rem;
}
.loading-spinner-large { font-size: 2.5rem; color: #c8f035; animation: spin 1s linear infinite; }
@keyframes spin { to { transform: rotate(360deg); } }
.no-data { text-align: center; padding: 3rem; color: #3d5470; }
.no-data i { font-size: 2rem; margin-bottom: 0.75rem; display: block; }

/* ── RESPONSIVE ── */
@media (max-width: 860px) {
  .main-grid {
    grid-template-columns: 1fr;
  }
  .left-col {
    position: static;
    flex-direction: row;
    flex-wrap: wrap;
  }
  .left-col .filter-panel { flex: 1; min-width: 200px; }
  .left-col .stat-card    { flex: 1; min-width: 180px; }
  .left-col .freq-panel   { width: 100%; }
  .athlete-table-wrap { max-height: 450px; }
}
@media (max-width: 600px) {
  .landing { padding: 1.5rem 1rem 3rem; }
  .gradient-text { font-size: 1.8rem; }
  .left-col {
    flex-direction: column;
  }
  .left-col .filter-panel,
  .left-col .stat-card,
  .left-col .freq-panel { width: 100%; flex: none; min-width: unset; }
  .stat-value { font-size: 2rem; }
  .panel-sub { display: none; }
  .bar-mini-track { display: none; }
  .athlete-table-wrap { max-height: 360px; }
  .athlete-table th:nth-child(4),
  .athlete-table td:nth-child(4) { display: none; }
  .panel { padding: 1.1rem; }
}
@media (max-width: 400px) {
  .gradient-text { font-size: 1.5rem; }
  .athlete-table th, .athlete-table td { padding: 8px 8px; font-size: 0.76rem; }
  .days-badge { padding: 2px 8px; font-size: 0.75rem; }
}
</style>