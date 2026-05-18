<template>
  <div class="landing">
    <div class="bg-animation">
      <div class="floating-shape shape-1"></div>
      <div class="floating-shape shape-2"></div>
      <div class="floating-shape shape-3"></div>
      <div class="floating-shape shape-4"></div>
    </div>

    <div class="hero-content">
      <div class="logo-container">
        <div class="logo">
          <i class="fas fa-bicycle"></i>
        </div>
        <div class="logo-glow"></div>
      </div>

      <h1 class="main-title">
        <span class="gradient-text">Bike to Work </span>
        <span class="highlight">Tracker</span>
      </h1>

      <p class="subtitle">
        <span class="typing-text">Track Your Journey, Save The Earth</span>
      </p>

      <p class="desc">
        Monitor your daily bike commuting activities, track your carbon savings,
        and contribute to a <span class="highlight-green">greener future</span> with Bike to Work.
      </p>

      <button class="cta" @click="goToDashboard">
        <span class="cta-text">
          <i class="fas fa-chart-line"></i>
          View Dashboard
        </span>
        <div class="cta-ripple"></div>
      </button>

      <div class="summary-section">
        <h2 class="summary-title">Overview</h2>

        <!-- TAB BUTTONS -->
        <div class="tab-selector">
          <button
            class="tab-btn"
            :class="{ 'active-summary': activeTab === 'summary' }"
            @click="switchTab('summary')"
          >
            <i class="fas fa-chart-bar"></i> Summary
          </button>
          <button
            class="tab-btn"
            :class="{ 'active-monthly': activeTab === 'monthly' }"
            @click="switchTab('monthly')"
          >
            <i class="fas fa-calendar-check"></i> Consistent BTW Rider
          </button>
        </div>

        <!-- TAB: SUMMARY -->
        <div v-if="activeTab === 'summary'" class="tab-content-box tab-content-summary">

          <div class="period-selector">
            <button class="period-btn period-month active" @click="changePeriod('month', $event)">Monthly</button>
            <button class="period-btn period-semester" @click="changePeriod('semester', $event)">Semester</button>
            <button class="period-btn period-year" @click="changePeriod('year', $event)">Yearly</button>
          </div>

          <!-- Monthly: year + month -->
          <div class="filter-row" v-if="currentPeriod === 'month'">
            <select v-model="selectedYear" @change="onSummaryYearChange" class="year-dropdown">
              <option v-for="y in availableYears" :key="y" :value="y">{{ y }}</option>
            </select>
            <select v-model="selectedMonth" @change="onMonthChange" class="month-dropdown">
              <option v-for="m in filteredSummaryMonths" :key="m.value" :value="m.value">
                {{ m.monthLabel }}
              </option>
            </select>
          </div>

          <!-- Semester -->
          <div class="filter-row" v-if="currentPeriod === 'semester'">
            <select v-model="selectedSemester" @change="onSemesterChange" class="semester-dropdown">
              <option value="1">Semester 1 (Jan – Jun {{ currentYear }})</option>
              <option value="2">Semester 2 (Jul – Dec {{ currentYear }})</option>
            </select>
          </div>

          <!-- Yearly -->
          <div class="filter-row" v-if="currentPeriod === 'year'">
            <select v-model="selectedYear" @change="onYearChange" class="year-dropdown">
              <option v-for="y in availableYears" :key="y" :value="y">{{ y }}</option>
            </select>
          </div>

          <div class="summary-grid">
            <div class="summary-card" v-for="(stat, index) in stats" :key="index">
              <div class="card-icon"><i :class="stat.icon"></i></div>
              <div class="card-value">
                <span v-if="loading" class="loading-spinner">⟳</span>
                <span v-else>{{ stat.value }}</span>
              </div>
              <div class="card-label">{{ stat.label }}</div>
              <div class="card-subtitle">{{ stat.subtitle }}</div>
            </div>
          </div>

          <div class="top-athletes-section">
            <h3 class="section-title">🏆 Top 20 Carbon Savers</h3>
            <div class="athletes-container">
              <div v-if="loadingAthletes" class="loading-container">
                <div class="loading-spinner-large">⟳</div>
                <p>Loading top commuter...</p>
              </div>
              <div v-else class="athletes-grid">
                <div
                  v-for="(athlete, index) in topAthletes"
                  :key="athlete.id || index"
                  class="athlete-card"
                  :class="{ 'top-three': index < 3 }"
                >
                  <div class="athlete-rank">
                    <span v-if="index === 0" class="crown gold">👑</span>
                    <span v-else-if="index === 1" class="crown silver">🥈</span>
                    <span v-else-if="index === 2" class="crown bronze">🥉</span>
                    <span v-else class="rank-number">#{{ index + 1 }}</span>
                  </div>
                  <div class="athlete-info">
                    <div class="athlete-name">{{ athlete.name || 'Anonymous' }}</div>
                    <div class="athlete-stats">
                      <div class="stat-item">
                        <i class="fas fa-leaf"></i>
                        <span class="carbon-saved">{{ formatCarbon(athlete.carbon_saved) }}</span>
                      </div>
                      <div class="stat-item">
                        <i class="fas fa-route"></i>
                        <span class="distance">{{ formatDistance(athlete.total_distance) }}</span>
                      </div>
                    </div>
                  </div>
                  <div class="athlete-badge" v-if="index < 3">
                    <i class="fas fa-star"></i>
                  </div>
                </div>
              </div>
              <div v-if="!loadingAthletes && topAthletes.length === 0" class="no-data">
                <i class="fas fa-info-circle"></i>
                <p>No commuter data available for this period</p>
              </div>
            </div>
          </div>

        </div>
        <!-- END TAB: SUMMARY -->

        <!-- TAB: MONTHLY ACTIVITY FREQUENCY -->
        <div v-if="activeTab === 'monthly'" class="tab-content-box tab-content-monthly monthly-tab">

          <div class="filter-row">
            <select v-model="monthlyFilterYear" @change="onMonthlyYearChange" class="year-dropdown">
              <option v-for="y in availableYears" :key="y" :value="y">{{ y }}</option>
            </select>
            <select v-model="monthlyFilterMonth" @change="onMonthlyFilterChange" class="month-dropdown">
              <option v-for="m in filteredMonthlyMonths" :key="m.value" :value="parseInt(m.value)">
                {{ m.monthLabel }}
              </option>
            </select>
          </div>

          <div v-if="loadingMonthlyFreq" class="loading-container">
            <div class="loading-spinner-large">⟳</div>
            <p>Loading...</p>
          </div>

          <div v-else>
            <!-- Frequency Distribution -->
            <div class="activity-chart" style="margin-bottom: 3rem;">
              <h3 class="chart-title">Monthly Activity Frequency (Mon–Fri)</h3>
              <div class="chart-bar" v-for="item in monthlyFreqDistribution" :key="item.label">
                <div class="bar-label">{{ item.label }}</div>
                <div class="bar-container">
                  <div class="bar-fill monthly-bar" :style="`width: ${item.percentage}%`"></div>
                </div>
                <div class="bar-value">{{ item.count }}</div>
              </div>
            </div>

            <!-- Athlete Attendance Table -->
            <h3 class="section-title" style="margin-top: 0;">🚴 Commuter List</h3>

            <div class="search-wrap">
              <i class="fas fa-search search-icon"></i>
              <input
                v-model="athleteSearch"
                type="text"
                placeholder="Search commuter name..."
                class="search-input"
                @input="onAthleteSearch"
              />
            </div>

            <p class="table-count" v-if="athleteSearch">
              Showing {{ filteredMonthlyAthletes.length }} of {{ monthlyAthletes.length }} athletes
            </p>

            <div v-if="filteredMonthlyAthletes.length === 0" class="no-data">
              <i class="fas fa-info-circle"></i>
              <p>No data available for this period</p>
            </div>

            <div v-else class="athlete-table-wrap">
              <table class="athlete-table">
                <thead>
                  <tr>
                    <th class="col-no">Rank</th>
                    <th class="col-name">Commuter Name</th>
                    <th class="col-center">Active Days</th>
                    <th class="col-center">Progress</th>
                    <th class="col-center">Total Distance</th>
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
                      <span v-if="getRank(athlete) === 0" class="crown gold">👑</span>
                      <span v-else-if="getRank(athlete) === 1" class="crown silver">🥈</span>
                      <span v-else-if="getRank(athlete) === 2" class="crown bronze">🥉</span>
                      <span v-else class="rank-number">#{{ getRank(athlete) + 1 }}</span>
                    </td>
                    <td class="col-name">{{ athlete.name }}</td>
                    <td class="col-center">
                      <span class="days-badge">{{ athlete.active_days }} days</span>
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
        <!-- END TAB: MONTHLY ACTIVITY FREQUENCY -->

      </div>
    </div>

    <div class="scroll-indicator">
      <div class="scroll-dot"></div>
      <div class="scroll-dot"></div>
      <div class="scroll-dot"></div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'LandingPage',
  data() {
    return {
      loading: true,
      loadingAthletes: true,
      loadingMonthlyFreq: false,
      activeTab: 'summary',
      currentPeriod: 'month',
      selectedYear: new Date().getFullYear(),
      currentYear: new Date().getFullYear(),
      selectedMonth: null,
      selectedSemester: '1',
      availableMonths: [],
      availableYears: [],
      filteredSummaryMonths: [],
      filteredMonthlyMonths: [],
      monthlyFilterMonth: new Date().getMonth() + 1,
      monthlyFilterYear: new Date().getFullYear(),
      athleteSearch: '',
      filteredMonthlyAthletes: [],
      stats: [
        { icon: 'fas fa-users',      value: '-', label: 'Total Participants', subtitle: 'Active cyclists',                key: 'total_participants'  },
        { icon: 'fas fa-bicycle',    value: '-', label: 'Total Rides',        subtitle: 'Cycling activities',             key: 'total_activities'    },
        { icon: 'fas fa-route',      value: '-', label: 'Total Distance',     subtitle: 'Kilometers covered',             key: 'total_distance'      },
        { icon: 'fas fa-clock',      value: '-', label: 'Total Time',         subtitle: 'Hours spent cycling',            key: 'total_time'          },
        { icon: 'fas fa-leaf',       value: '-', label: 'CO₂ Saved',          subtitle: 'Kilograms of emissions avoided', key: 'total_carbon_saved'  },
        { icon: 'fas fa-chart-line', value: '-', label: 'Daily Average',      subtitle: 'Kilometers per day',             key: 'daily_average'       }
      ],
      topAthletes: [],
      monthlyFreqDistribution: [
        { label: '1–4 days',   percentage: 0, count: '-', key: '1_to_4_days'   },
        { label: '5–9 days',   percentage: 0, count: '-', key: '5_to_9_days'   },
        { label: '10–14 days', percentage: 0, count: '-', key: '10_to_14_days' },
        { label: '15–19 days', percentage: 0, count: '-', key: '15_to_19_days' },
        { label: '20+ days',   percentage: 0, count: '-', key: '20_plus_days'  }
      ],
      monthlyAthletes: [],
      apiBaseUrl: process.env.VUE_APP_API_URL 
      //|| 'https://carbonsaved.bike2work.id/api'
    }
  },

  async mounted() {
    const now = new Date()
    this.selectedMonth = (now.getMonth() + 1).toString()
    this.selectedYear  = this.currentYear
    await this.fetchAvailableMonths()
    await this.fetchSummaryData()
    await this.fetchTopAthletes()
  },

  methods: {
    // ===== FORMAT HELPERS =====
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
    formatCarbon(carbon) {
      if (!carbon) return '0 kg'
      const v = typeof carbon === 'string' ? parseFloat(carbon) : carbon
      return v >= 1000
        ? `${this.formatNumber(v)} kg`
        : `${this.formatNumberWithDecimals(v, 1)} kg`
    },
    formatTimeHours(seconds) {
      if (!seconds) return '0 hrs'
      const hrs = Math.round((typeof seconds === 'string' ? parseInt(seconds) : seconds) / 3600)
      return `${this.formatNumber(hrs)} hrs`
    },
    getActiveDayPercent(activeDays) {
      if (!this.monthlyAthletes.length) return 0
      const max = this.monthlyAthletes[0].active_days
      return max > 0 ? Math.round((activeDays / max) * 100) : 0
    },
    getRank(athlete) {
      return this.monthlyAthletes.findIndex(a => a.id === athlete.id)
    },

    // ===== SEARCH =====
    onAthleteSearch() {
      const q = this.athleteSearch.toLowerCase().trim()
      this.filteredMonthlyAthletes = q
        ? this.monthlyAthletes.filter(a => a.name.toLowerCase().includes(q))
        : [...this.monthlyAthletes]
    },

    // ===== MONTH LABEL HELPER =====
    toMonthLabel(year, monthVal) {
      return new Date(year, parseInt(monthVal) - 1, 1)
        .toLocaleString('en-US', { month: 'long' })
    },

    // ===== API METHODS =====
    async fetchAvailableMonths() {
      try {
        const response = await axios.get(`${this.apiBaseUrl}/available-periods`)
        if (response.data?.status === 200 && response.data.data) {
          this.availableMonths = response.data.data.months.map(item => ({
            value: item.month.toString(),
            label: new Date(item.year, item.month - 1, 1)
              .toLocaleString('en-US', { month: 'long', year: 'numeric' }),
            year: item.year
          }))
          this.availableYears = response.data.data.years || []

          const now       = new Date()
          const currYear  = now.getFullYear()
          const currMonth = (now.getMonth() + 1).toString()

          // ── Summary filter init ──
          this.selectedYear = this.availableYears.includes(currYear)
            ? currYear
            : (this.availableYears[0] || currYear)

          this.filteredSummaryMonths = this.availableMonths
            .filter(m => m.year === this.selectedYear)
            .map(m => ({ ...m, monthLabel: this.toMonthLabel(m.year, m.value) }))

          const foundSummary = this.filteredSummaryMonths.find(m => m.value === currMonth)
          this.selectedMonth = foundSummary
            ? currMonth
            : (this.filteredSummaryMonths[0]?.value || currMonth)

          // ── Monthly tab filter init ──
          this.monthlyFilterYear = this.availableYears.includes(currYear)
            ? currYear
            : (this.availableYears[0] || currYear)

          this.filteredMonthlyMonths = this.availableMonths
            .filter(m => m.year === this.monthlyFilterYear)
            .map(m => ({ ...m, monthLabel: this.toMonthLabel(m.year, m.value) }))

          const foundMonthly = this.filteredMonthlyMonths.find(
            m => parseInt(m.value) === parseInt(currMonth)
          )
          this.monthlyFilterMonth = foundMonthly
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

    async fetchTopAthletes() {
      try {
        this.loadingAthletes = true
        let params = {}
        if (this.currentPeriod === 'month') {
          params = {
            period: 'month',
            month:  parseInt(this.selectedMonth),
            year:   this.selectedYear
          }
        } else if (this.currentPeriod === 'semester') {
          params = { period: 'year', year: this.currentYear }
        } else {
          params = { period: 'year', year: this.selectedYear }
        }
        const response = await axios.get(`${this.apiBaseUrl}/total-athlete`, { params })
        if (response.data?.status === 200) {
          this.topAthletes = (response.data.data || []).map(a => ({
            id:               a.id_athlete,
            name:             [a.athlete_firstname, a.athlete_lastname].filter(Boolean).join(' ') || 'Anonymous',
            carbon_saved:     a.total_carbon_saving,
            total_distance:   a.total_distance,
            total_activities: a.total_activity,
            total_time:       a.total_time
          })).slice(0, 20)
        } else {
          this.topAthletes = []
        }
      } catch (error) {
        console.error('Error fetching top athletes:', error)
        this.topAthletes = []
      } finally {
        this.loadingAthletes = false
      }
    },

    async fetchSummaryData() {
      try {
        this.loading = true
        let params = {}
        if (this.currentPeriod === 'month') {
          params = {
            period: 'month',
            month:  parseInt(this.selectedMonth),
            year:   this.selectedYear
          }
        } else if (this.currentPeriod === 'semester') {
          params = { period: 'semester', year: this.currentYear, semester: this.selectedSemester }
        } else {
          params = { period: 'year', year: this.selectedYear }
        }
        const response = await axios.get(`${this.apiBaseUrl}/summary-stats`, { params })
        if (response.data?.status === 200) this.updateStatsFromAPI(response.data.data)
      } catch (error) {
        console.error('Error fetching summary data:', error)
      } finally {
        this.loading = false
      }
    },

    async fetchMonthlyActivityFrequency() {
      try {
        this.loadingMonthlyFreq = true
        const response = await axios.get(`${this.apiBaseUrl}/monthly-activity-frequency`, {
          params: { month: this.monthlyFilterMonth, year: this.monthlyFilterYear }
        })
        if (response.data?.status === 200) {
          const { athletes, frequency_distribution } = response.data.data

          this.monthlyAthletes = athletes.map(a => ({
            id:             a.id_athlete,
            name:           [a.athlete_firstname, a.athlete_lastname].filter(Boolean).join(' ') || 'Anonymous',
            active_days:    parseInt(a.active_days) || 0,
            total_distance: a.total_distance
          }))

          this.athleteSearch           = ''
          this.filteredMonthlyAthletes = [...this.monthlyAthletes]

          const dist  = frequency_distribution
          const total = Object.values(dist).reduce((sum, v) => sum + parseInt(v || 0), 0)
          this.monthlyFreqDistribution.forEach(item => {
            const count     = parseInt(dist[item.key] || 0)
            item.count      = this.formatNumber(count)
            item.percentage = total > 0 ? Math.max(Math.round((count / total) * 100), count > 0 ? 5 : 0) : 0
          })
        }
      } catch (err) {
        console.error('Error fetching monthly activity frequency:', err)
      } finally {
        this.loadingMonthlyFreq = false
      }
    },

    updateStatsFromAPI(data) {
      const daysInPeriod = this.currentPeriod === 'month'
        ? new Date(this.selectedYear, parseInt(this.selectedMonth), 0).getDate()
        : 180
      const totalDist  = typeof data.total_distance === 'string' ? parseFloat(data.total_distance) : data.total_distance
      const dailyAvgKm = totalDist ? (totalDist / 1000 / daysInPeriod) : 0

      this.stats.forEach(stat => {
        try {
          switch (stat.key) {
            case 'total_participants': stat.value = this.formatNumber(data.total_participants);  break
            case 'total_activities':   stat.value = this.formatNumber(data.total_activities);   break
            case 'total_distance':     stat.value = this.formatDistance(data.total_distance);   break
            case 'total_time':         stat.value = this.formatTimeHours(data.total_time);      break
            case 'total_carbon_saved': stat.value = this.formatCarbon(data.total_carbon_saved); break
            case 'daily_average':
              stat.value = dailyAvgKm >= 1000
                ? `${this.formatNumber(dailyAvgKm)} km`
                : `${this.formatNumberWithDecimals(dailyAvgKm, 1)} km`
              break
          }
        } catch (e) { stat.value = '-' }
      })
      this.$forceUpdate()
    },

    // ===== EVENT HANDLERS =====
    async switchTab(tab) {
      this.activeTab = tab
      if (tab === 'monthly' && this.monthlyAthletes.length === 0) {
        await this.fetchMonthlyActivityFrequency()
      }
    },

    onSummaryYearChange() {
      this.filteredSummaryMonths = this.availableMonths
        .filter(m => m.year === this.selectedYear)
        .map(m => ({ ...m, monthLabel: this.toMonthLabel(m.year, m.value) }))
      if (this.filteredSummaryMonths.length > 0) {
        this.selectedMonth = this.filteredSummaryMonths[0].value
      }
      this.fetchSummaryData()
      this.fetchTopAthletes()
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
    },
    async onSemesterChange() {
      await this.fetchSummaryData()
      await this.fetchTopAthletes()
    },
    async onMonthChange() {
      await this.fetchSummaryData()
      await this.fetchTopAthletes()
    },
    async onYearChange() {
      await this.fetchSummaryData()
      await this.fetchTopAthletes()
    },

    async changePeriod(period, event) {
      document.querySelectorAll('.period-btn').forEach(btn => btn.classList.remove('active'))
      event.target.classList.add('active')
      this.currentPeriod = period
      await this.fetchSummaryData()
      await this.fetchTopAthletes()
    },

    goToDashboard() {
      const ripple = event.currentTarget.querySelector('.cta-ripple')
      ripple.style.animation = 'ripple 0.6s ease-out'
      setTimeout(() => { ripple.style.animation = '' }, 600)
      document.body.style.transition = 'opacity 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94)'
      document.body.style.opacity    = '0.8'
      setTimeout(() => {
        this.$router.push('/Dashboard')
        document.body.style.opacity = '1'
      }, 500)
    }
  }
}
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }

.landing {
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #224d7e 0%, #0c2540 50%, #133a5a 100%);
  color: #ffffff;
  text-align: center;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow-x: hidden;
}

/* ===== BG ANIMATION ===== */
.bg-animation { position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 0; }
.floating-shape { position: absolute; border-radius: 50%; background: rgba(255,255,255,0.1); animation: float 6s ease-in-out infinite; }
.shape-1 { width: 80px;  height: 80px;  top: 10%; left: 10%;  animation-delay: 0s; }
.shape-2 { width: 120px; height: 120px; top: 60%; right: 10%; animation-delay: 2s; }
.shape-3 { width: 60px;  height: 60px;  bottom: 30%; left: 20%; animation-delay: 4s; }
.shape-4 { width: 100px; height: 100px; top: 30%; right: 30%; animation-delay: 1s; }
@keyframes float { 0%, 100% { transform: translateY(0px) rotate(0deg); } 50% { transform: translateY(-20px) rotate(180deg); } }

/* ===== HERO ===== */
.hero-content { position: relative; z-index: 1; animation: fadeInUp 1s ease-out forwards; }
@keyframes fadeInUp { from { opacity: 0; transform: translateY(30px); } to { opacity: 1; transform: translateY(0); } }
.logo-container { position: relative; margin-bottom: 2rem; }
.logo { width: 120px; height: 120px; background: #18cf55; border-radius: 50%; display: flex; align-items: center; justify-content: center; margin: 0 auto; animation: logoFloat 3s ease-in-out infinite; z-index: 2; position: relative; box-shadow: 0 20px 40px rgba(24,207,85,0.3); }
.logo i { font-size: 3rem; color: white; }
.logo-glow { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 150px; height: 150px; background: radial-gradient(circle, rgba(24,207,85,0.3) 0%, transparent 70%); border-radius: 50%; animation: pulse 2s ease-in-out infinite; }
@keyframes logoFloat { 0%, 100% { transform: translateY(0px); } 50% { transform: translateY(-10px); } }
@keyframes pulse { 0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.6; } 50% { transform: translate(-50%, -50%) scale(1.1); opacity: 0.8; } }
.main-title { font-size: 3.5rem; font-weight: 900; margin-bottom: 1rem; line-height: 1.2; }
.gradient-text { background: linear-gradient(45deg, #18cf55, #18cf55); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
.highlight { color: #ffffff; text-shadow: 0 0 20px rgba(255,255,255,0.5); }
.highlight-green { color: #18cf55; font-weight: 600; }
.subtitle { font-size: 1.8rem; margin-bottom: 1.5rem; color: #e0e0e0; min-height: 2.5rem; }
.typing-text { display: inline-block; border-right: 2px solid #60a5fa; animation: typing 3s steps(30, end) 1 forwards, blink 0.8s step-end 5; }
@keyframes typing { from { width: 0; } to { width: 100%; } }
@keyframes blink { 50% { border-color: transparent; } }
.desc { max-width: 600px; margin: 0 auto 3rem; font-size: 1.1rem; line-height: 1.6; color: #e0e0e0; opacity: 0.9; }

/* ===== CTA ===== */
.cta { background: linear-gradient(45deg, #149e42, #18cf55); color: #ffffff; font-weight: 700; padding: 1.2rem 3rem; border-radius: 50px; border: none; font-size: 1.1rem; cursor: pointer; position: relative; overflow: hidden; transition: all 0.3s; box-shadow: 0 15px 35px rgba(24,207,85,0.3); margin-top: 2rem; }
.cta:hover { transform: translateY(-5px); box-shadow: 0 20px 50px rgba(24,207,85,0.4); }
.cta-ripple { position: absolute; top: 50%; left: 50%; width: 0; height: 0; border-radius: 50%; background: rgba(255,255,255,0.3); transform: translate(-50%, -50%); }
@keyframes ripple { to { width: 300px; height: 300px; opacity: 0; } }

/* ===== SUMMARY SECTION ===== */
.summary-section { margin: 4rem 0; }
.summary-title { font-size: 2.5rem; font-weight: 800; margin-bottom: 2rem; background: linear-gradient(45deg, #60a5fa, #18cf55); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
.section-title { font-size: 2rem; font-weight: 700; margin: 4rem 0 2rem; background: linear-gradient(45deg, #ffd700, #ff6b35); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }

/* ===== TAB CONTENT BOX ===== */
.tab-content-box {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 24px;
  padding: 2rem;
  margin-top: 1.5rem;
  backdrop-filter: blur(12px);
  animation: fadeIn 0.3s ease-out;
}
.tab-content-summary { border-top: 3px solid #378ADD; }
.tab-content-monthly { border-top: 3px solid #378ADD; }

/* ===== TABS ===== */
.tab-selector { display: flex; justify-content: center; gap: 1rem; margin-bottom: 0.5rem; flex-wrap: wrap; }
.tab-btn { padding: 0.8rem 2rem; background: rgba(255,255,255,0.1); border: 2px solid rgba(255,255,255,0.2); border-radius: 30px; color: #e0e0e0; cursor: pointer; transition: all 0.3s ease; font-weight: 600; backdrop-filter: blur(10px); }
.tab-btn.active-summary { background: linear-gradient(45deg, #378ADD, #185FA5); border-color: #378ADD; color: white; box-shadow: 0 10px 30px rgba(24,207,85,0.3); transform: translateY(-2px); }
.tab-btn.active-monthly { background: linear-gradient(45deg, #378ADD, #185FA5); border-color: #378ADD; color: white; box-shadow: 0 10px 30px rgba(55,138,221,0.3); transform: translateY(-2px); }
.tab-btn:not(.active-summary):not(.active-monthly):hover { background: rgba(255,255,255,0.2); border-color: rgba(255,255,255,0.4); transform: translateY(-2px); }

/* ===== PERIOD BUTTONS ===== */
.period-selector { display: flex; justify-content: center; gap: 1rem; margin-bottom: 2rem; flex-wrap: wrap; }
.period-btn { padding: 0.8rem 2rem; background: rgba(255,255,255,0.1); border: 2px solid rgba(255,255,255,0.2); color: #e0e0e0; cursor: pointer; transition: all 0.3s ease; font-weight: 600; backdrop-filter: blur(10px); }
.period-month    { border-radius: 12px; }
.period-semester { border-radius: 12px; }
.period-year     { border-radius: 12px;  }
.period-btn.active.period-month    { background: linear-gradient(45deg, #378ADD, #185FA5); border-color: #378ADD; color: white; box-shadow: 0 10px 30px rgba(24,207,85,0.3); transform: translateY(-2px); }
.period-btn.active.period-semester { background: linear-gradient(45deg, #378ADD, #185FA5); border-color: #378ADD; color: white; box-shadow: 0 10px 30px rgba(245,158,11,0.3); transform: translateY(-2px); }
.period-btn.active.period-year     { background: linear-gradient(45deg, #378ADD, #185FA5); border-color: #378ADD; color: white; box-shadow: 0 10px 30px rgba(167,139,250,0.3); transform: translateY(-2px); }
.period-btn:not(.active):hover { background: rgba(255,255,255,0.2); border-color: rgba(255,255,255,0.4); transform: translateY(-2px); }

/* ===== FILTER ROW ===== */
.filter-row { display: flex; justify-content: center; gap: 1rem; margin-bottom: 2rem; flex-wrap: wrap; animation: fadeIn 0.3s ease-out; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(-10px); } to { opacity: 1; transform: translateY(0); } }

/* ===== DROPDOWNS ===== */
.month-dropdown, .semester-dropdown, .year-dropdown { padding: 0.8rem 1.5rem; background: rgba(255,255,255,0.1); border: 2px solid rgba(255,255,255,0.2); border-radius: 15px; color: #e0e0e0; font-size: 1rem; font-weight: 500; cursor: pointer; transition: all 0.3s ease; backdrop-filter: blur(10px); min-width: 160px; }
.month-dropdown:focus, .semester-dropdown:focus, .year-dropdown:focus { outline: none; border-color: #378ADD; box-shadow: 0 0 0 3px rgba(24,207,85,0.2); }
.month-dropdown option, .semester-dropdown option, .year-dropdown option { background: #133a5a; color: #e0e0e0; }

/* ===== SUMMARY GRID ===== */
.summary-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 2rem; max-width: 1200px; margin: 0 auto 3rem; }
.summary-card { background: rgba(255,255,255,0.1); backdrop-filter: blur(20px); border: 1px solid rgba(255,255,255,0.2); border-radius: 20px; padding: 2rem; transition: all 0.3s cubic-bezier(0.25,0.46,0.45,0.94); }
.summary-card:hover { transform: translateY(-10px) scale(1.02); box-shadow: 0 30px 60px rgba(0,0,0,0.3); }
.card-icon { font-size: 3rem; margin-bottom: 1rem; background: linear-gradient(45deg, #60a5fa, #18cf55); -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text; }
.card-value { font-size: 2.5rem; font-weight: 800; color: #18cf55; margin-bottom: 0.5rem; }
.card-label { font-size: 1.1rem; color: #e0e0e0; font-weight: 500; }
.card-subtitle { font-size: 0.9rem; color: #b0b0b0; margin-top: 0.5rem; }

/* ===== TOP ATHLETES ===== */
.top-athletes-section { margin-top: 4rem; max-width: 1200px; margin-left: auto; margin-right: auto; }
.athletes-container { width: 100%; }
.athletes-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 1.5rem; margin-top: 2rem; max-width: 800px; margin-left: auto; margin-right: auto; }
.athlete-card { background: rgba(255,255,255,0.1); backdrop-filter: blur(20px); border: 1px solid rgba(255,255,255,0.2); border-radius: 15px; padding: 1.5rem; display: flex; align-items: center; gap: 1rem; transition: all 0.3s; position: relative; overflow: hidden; }
.athlete-card:hover { transform: translateY(-5px) scale(1.02); box-shadow: 0 20px 40px rgba(0,0,0,0.2); }
.athlete-card.top-three { border: 2px solid #ffd700; background: linear-gradient(135deg, rgba(255,215,0,0.1), rgba(255,255,255,0.1)); }
.athlete-rank { display: flex; align-items: center; justify-content: center; width: 50px; height: 50px; border-radius: 50%; background: rgba(255,255,255,0.1); flex-shrink: 0; }
.crown { font-size: 1.8rem; }
.crown.gold   { filter: drop-shadow(0 0 10px #ffd700); }
.crown.silver { filter: drop-shadow(0 0 10px #c0c0c0); }
.crown.bronze { filter: drop-shadow(0 0 10px #cd7f32); }
.rank-number { font-size: 1.2rem; font-weight: 700; color: #18cf55; }
.athlete-info { flex: 1; }
.athlete-name { font-size: 1.1rem; font-weight: 600; color: #ffffff; margin-bottom: 0.5rem; text-align: left; }
.athlete-stats { display: flex; gap: 1rem; flex-wrap: wrap; }
.stat-item { display: flex; align-items: center; gap: 0.5rem; font-size: 0.9rem; color: #e0e0e0; }
.stat-item i { color: #18cf55; width: 16px; }
.carbon-saved { color: #18cf55; font-weight: 600; }
.distance { color: #60a5fa; font-weight: 600; }
.athlete-badge { position: absolute; top: 10px; right: 10px; color: #ffd700; font-size: 1.2rem; animation: sparkle 2s ease-in-out infinite; }
@keyframes sparkle { 0%, 100% { transform: scale(1) rotate(0deg); opacity: 0.8; } 50% { transform: scale(1.2) rotate(180deg); opacity: 1; } }

/* ===== ACTIVITY CHART ===== */
.activity-chart { max-width: 600px; margin: 0 auto; }
.chart-title { font-size: 1.5rem; margin-bottom: 2rem; color: #e0e0e0; }
.chart-bar { display: flex; align-items: center; margin-bottom: 1rem; gap: 1rem; }
.bar-label { width: 140px; text-align: left; font-size: 0.9rem; color: #e0e0e0; }
.bar-container { flex: 1; height: 20px; background: rgba(255,255,255,0.1); border-radius: 10px; overflow: hidden; }
.bar-fill { height: 100%; background: linear-gradient(45deg, #378ADD, #378ADD); border-radius: 10px; transition: width 0.8s ease; }
.monthly-bar { background: linear-gradient(45deg, #378ADD, #185FA5) !important; }
.bar-value { width: 40px; text-align: right; font-weight: 600; color: #60a5fa; }

/* ===== MONTHLY TAB ===== */
.monthly-tab { margin-top: 1.5rem; }

/* ===== SEARCH ===== */
.search-wrap { position: relative; max-width: 400px; margin: 0 auto 1.5rem; }
.search-icon { position: absolute; left: 1rem; top: 50%; transform: translateY(-50%); color: rgba(255,255,255,0.4); font-size: 0.9rem; pointer-events: none; }
.search-input { width: 100%; padding: 0.8rem 1rem 0.8rem 2.8rem; background: rgba(255,255,255,0.1); border: 2px solid rgba(255,255,255,0.2); border-radius: 30px; color: #e0e0e0; font-size: 1rem; transition: all 0.3s; backdrop-filter: blur(10px); }
.search-input:focus { outline: none; border-color: f#378ADD; box-shadow: 0 0 0 3px rgba(55,138,221,0.2); }
.search-input::placeholder { color: rgba(255,255,255,0.4); }
.table-count { margin-bottom: 1rem; font-size: 0.85rem; color: #b0b0b0; text-align: center; }

/* ===== ATHLETE TABLE ===== */
.athlete-table-wrap { max-height: 460px; overflow-y: auto; overflow-x: auto; border-radius: 15px; border: 1px solid rgba(255,255,255,0.15); max-width: 900px; margin: 0 auto; scrollbar-width: thin; scrollbar-color: rgba(55,138,221,0.5) rgba(255,255,255,0.05); }
.athlete-table-wrap::-webkit-scrollbar { width: 6px; height: 6px; }
.athlete-table-wrap::-webkit-scrollbar-track { background: rgba(255,255,255,0.05); border-radius: 3px; }
.athlete-table-wrap::-webkit-scrollbar-thumb { background: rgba(55,138,221,0.5); border-radius: 3px; }
.athlete-table-wrap::-webkit-scrollbar-thumb:hover { background: rgba(55,138,221,0.8); }
.athlete-table { width: 100%; border-collapse: collapse; font-size: 0.9rem; }
.athlete-table thead tr { position: sticky; top: 0; z-index: 2; }
.athlete-table th { padding: 14px 16px; color: #e0e0e0; font-weight: 600; border-bottom: 1px solid rgba(255,255,255,0.15); white-space: nowrap; background: rgba(20,50,80,0.97); }
.athlete-table td { padding: 12px 16px; color: #e0e0e0; border-bottom: 0.5px solid rgba(255,255,255,0.08); vertical-align: middle; }
.athlete-table tbody tr:last-child td { border-bottom: none; }
.athlete-table tbody tr:hover { background: rgba(255,255,255,0.06); transition: background 0.2s; }
.row-gold   { background: rgba(255,215,0,0.08)   !important; }
.row-silver { background: rgba(192,192,192,0.06) !important; }
.row-bronze { background: rgba(205,127,50,0.06)  !important; }
.col-no     { text-align: center; width: 64px; }
.col-name   { font-weight: 600; color: #ffffff; text-align: left; }
.col-center { text-align: center; }
.days-badge { display: inline-block; background: rgba(24,207,85,0.15); color: #378ADD; border: 1px solid rgba(24,207,85,0.3); border-radius: 20px; padding: 3px 14px; font-weight: 600; font-size: 0.85rem; white-space: nowrap; }
.bar-mini-track { width: 80px; height: 8px; background: rgba(255,255,255,0.1); border-radius: 4px; overflow: hidden; display: inline-block; vertical-align: middle; }
.bar-mini-fill  { height: 100%; border-radius: 4px; background: linear-gradient(45deg, #378ADD, #378ADD); transition: width 0.6s ease; }
.km-txt { color: #85B7EB; font-weight: 600; }

/* ===== LOADING ===== */
.loading-container { display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 3rem; color: #e0e0e0; }
.loading-spinner       { display: inline-block; animation: spin 1s linear infinite; font-size: 1.5rem; color: #18cf55; }
.loading-spinner-large { font-size: 3rem; color: #18cf55; animation: spin 1s linear infinite; margin-bottom: 1rem; }
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
.no-data { text-align: center; padding: 3rem; color: #b0b0b0; }
.no-data i { font-size: 3rem; margin-bottom: 1rem; opacity: 0.5; }

/* ===== SCROLL INDICATOR ===== */
.scroll-indicator { position: fixed; right: 2rem; top: 50%; transform: translateY(-50%); display: flex; flex-direction: column; gap: 1rem; }
.scroll-dot { width: 12px; height: 12px; border-radius: 50%; background: rgba(255,255,255,0.3); }
.scroll-dot:first-child { background: #18cf55; }

/* ===== RESPONSIVE ===== */
@media (max-width: 768px) {
  .main-title { font-size: 2.5rem; }
  .subtitle { font-size: 1.3rem; }
  .summary-grid { grid-template-columns: 1fr; gap: 1.5rem; }
  .athletes-grid { grid-template-columns: 1fr; gap: 1rem; max-width: 500px; margin: 2rem auto 0; }
  .period-selector { flex-direction: column; align-items: center; }
  .tab-selector { flex-direction: column; align-items: center; }
  .filter-row { flex-direction: column; align-items: center; }
  .bar-label { width: 120px; font-size: 0.9rem; }
  .scroll-indicator { display: none; }
  .athlete-stats { flex-direction: column; gap: 0.5rem; }
  .bar-mini-track { width: 50px; }
  .athlete-table th, .athlete-table td { padding: 10px 12px; }
  .tab-content-box { padding: 1.2rem; }

  .athlete-table-wrap { max-width: 100%; border-radius: 10px; }
  .athlete-table { font-size: 0.8rem; }
  .col-name { max-width: 120px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .bar-mini-track { display: none; } /* sembunyiin progress bar, terlalu sempit */
  .days-badge { padding: 3px 10px; font-size: 0.78rem; }
  .search-wrap { max-width: 100%; }
  .chart-bar { gap: 0.5rem; }
  .bar-label { width: 100px; font-size: 0.8rem; }
  .activity-chart { max-width: 100%; padding: 0 0.5rem; }
  .chart-title { font-size: 1.2rem; }
  .section-title { font-size: 1.5rem; }
}
@media (max-width: 480px) {
  .landing { padding: 1rem; }
  .main-title { font-size: 2rem; }
  .summary-card { padding: 1.5rem; }
  .athlete-table th, .athlete-table td { padding: 8px 10px; font-size: 0.8rem; }
  .tab-btn { padding: 0.7rem 1.2rem; font-size: 0.9rem; }
  .search-input { font-size: 0.9rem; }

  .athlete-table th, .athlete-table td { padding: 8px 8px; font-size: 0.75rem; }
  .col-no { width: 44px; }
  .col-name { max-width: 100px; }
  .km-txt { font-size: 0.75rem; }
  .days-badge { padding: 2px 8px; font-size: 0.72rem; }
  .month-dropdown, .year-dropdown { min-width: 130px; font-size: 0.9rem; padding: 0.6rem 1rem; }
}
</style>