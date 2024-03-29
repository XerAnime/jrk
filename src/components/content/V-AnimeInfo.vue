<template>
  <div class="container mx-auto">
    <template v-if="isLoading">
      <VSpinner />
    </template>
    <template v-else-if="anime">
      <div v-if="anime" class="max-w-screen-2xl mx-auto p-6">
        <nav class="flex py-5" aria-label="Breadcrumb">
          <ol class="inline-flex items-center">
            <li class="inline-flex items-center">
              <div class="flex items-center">
                <RouterLink to="/">
                  <p class="text-xs hover:underline hover:text-orange-200">Home</p>
                </RouterLink>
              </div>
            </li>
            <li>
              <div class="flex items-center">
                <div i-carbon-chevron-right class="text-sm mx-1" />
                <RouterLink to="/home">
                  <p class="text-xs hover:underline hover:text-orange-200">Anime</p>
                </RouterLink>
              </div>
            </li>
            <li aria-current="page">
              <div class="flex items-center">
                <div i-carbon-chevron-right class="text-sm mx-1" />
                <p class="text-xs">{{ anime.title?.english || anime.title?.userPreferred || anime.title?.romaji }}</p>
              </div>
            </li>
          </ol>
        </nav>

        <div class="grid grid-cols-1 gap-6 md:grid-cols-3 lg:grid-cols-3 animated animated-fade-in">
          <div class="col-span-1 md:col-span-2 overflow-hidden">
            <div class="relative w-full">
              <div v-if="anime.type === 'MUSIC'" class="video-container">
                <iframe
                  class="w-full h-60 md:h-140"
                  :src="`https://www.youtube.com/embed/${anime.trailer.id}?autoplay=0&controls=1&showinfo=0&rel=0&modestbranding=1&loop=0&fs=0&playsinline=1&mute=0&playlist=${anime.trailer.id}`"
                  frameborder="1"
                  allowfullscreen
                ></iframe>
              </div>
              <div v-else>
                <video
                  ref="videoElement"
                  id="video-element"
                  class="video-js w-full h-60 md:h-140 z-10 py-1 px-2"
                  controls
                  preload="auto"
                  width="800"
                  height="500"
                  :poster="anime.cover"
                  data-setup="{}"
                >
                  <template v-if="anime.type === 'MUSIC'">
                    <source :src="anime.trailer.thumbnail" type="video/mp4" />
                  </template>
                </video>
              </div>
            </div>

            <div v-if="anime && anime.type !== 'MUSIC'" class="flex items-center py-3 pt-2">
              <div class="flex items-center">
                <span class="mr-1 text-sm">Episodes</span>
                <select
                  v-model="selectedRange"
                  @change="updateDisplayedEpisodes"
                  class="btn px-2 py-1 mr-2 border border-dark-300"
                >
                  <option v-if="episodeRanges.length === 0" value="0">0</option>
                  <option v-else v-for="range in episodeRanges" :key="range" :value="range">
                    {{ range }}
                  </option>
                </select>
              </div>
              <div class="flex">
                <button
                  class="btn text-xs flex items-center border border-dark-300 mr-1"
                  @click="selectPreviousEpisode"
                >
                  <div i-mdi-skip-backward class="text-sm mr-2" />
                  <span class="uppercase">Previous</span>
                </button>
                <button
                  class="btn text-xs flex items-center border border-dark-300 ml-1"
                  @click="selectNextEpisode"
                >
                  <div i-mdi-skip-forward class="text-sm mr-2" />
                  <span class="uppercase">Next</span>
                </button>
              </div>
            </div>

            <div v-if="anime && anime.type !== 'MUSIC'" class="grid grid-cols-6 md:grid-cols-9 lg:grid-cols-11 xl:grid-cols-12 gap-1 px-5">
              <div
                v-for="(episode, index) in displayedEpisodes"
                :key="index"
                :class="[
                  'text-xs',
                  'text-center',
                  'bg-dark-200',
                  'p-1.5',
                  'border',
                  'border-dark-100',
                  'hover:bg-dark-50',
                  'cursor-pointer',
                  { 'active-episode': episode === activeEpisode }
                ]"
                @click="selectEpisode(episode)"
              >
                <p>{{ episode.number }}</p>
              </div>
            </div>

            <div class="py-5">
              <div
                v-if="filteredRelations.length > 0"
                class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-6 gap-4"
              >
                <RouterLink
                  :to="'/anime/' + relation.id"
                  v-for="(relation, index) in filteredRelations"
                  :key="index"
                  class="btn relative group border border-dark-300"
                >
                  <img
                    :src="relation.image"
                    draggable="false"
                    class="w-full h-10 object-cover opacity-10"
                    alt="Anime Cover"
                  />
                  <div class="absolute inset-0 flex items-center justify-center text-center">
                    <p class="text-sm font-semibold">{{ relation.relationType }}</p>
                  </div>
                </RouterLink>
              </div>
            </div>
          </div>

          <div class="col-span-1 container">
            <div class="flex-col md:flex-row lg:items-start">
              <div class="items-center justify-center mx-auto text-center">
                <img
                  :src="anime.image"
                  draggable="false"
                  class="w-75 pt-5 object-cover mb-4 md:mr-4 px-2"
                />
              </div>
              <div class="w-full">
                <div class="flex items-center">
                  <h1 class="text-2xl font-bold mr-2 w-80">
                    {{ anime.title?.english || anime.title?.userPreferred || anime.title?.romaji }}
                  </h1>
                </div>
                <div class="description-container w-75 mb-4">
                  <p
                    class="text-sm"
                    :class="{ truncate: !showFullDescription }"
                    v-html="parseDescription(anime.description)"
                  ></p>
                </div>
                <div class="mb-4">
                  <span
                    v-if="anime.description.length > 150 && !this.showFullDescription"
                    class="font-bold cursor-pointer hover:underline"
                    @click="this.showFullDescription = true"
                    >View more</span
                  >
                  <span
                    v-if="this.showFullDescription"
                    class="font-bold cursor-pointer hover:underline"
                    @click="this.showFullDescription = false"
                    >View less</span
                  >
                </div>
                <div class="items-center">
                  <div class="flex items-center mb-2">
                    <div i-openmoji-star class="mr-1 py-2"></div>
                    <span>{{ (anime.rating / 10).toFixed(1) }}</span>
                  </div>
                  <p class="text-sm mr-2">Studio: {{ anime.studios[0] }}</p>
                  <div>
                    <p class="text-sm mr-2">
                      Type:
                      <RouterLink
                        class="hover:underline hover:text-orange-300"
                        :to="'/type/' + anime.type"
                        >{{ anime.type }}</RouterLink
                      >
                    </p>
                  </div>
                  <p class="text-sm mr-2">
                    Alternate Title: {{ anime.title?.native || anime.title?.romaji }}
                  </p>
                  <p class="text-sm mr-2">Status: {{ anime.status }}</p>
                  <p class="text-sm mr-2">
                    Sub/Dub: <span class="uppercase">{{ anime.subOrDub }}</span>
                  </p>
                  <p class="text-sm mr-2">Country: {{ anime.countryOfOrigin }}</p>
                  <div class="text-sm mr-2 flex items-center">
                    <span>Genres:</span>
                    <p v-for="(genre, index) in anime.genres" :key="index">
                      <RouterLink
                        :to="`/genre/${genre}`"
                        class="ml-1 hover:underline hover:text-orange-300"
                        >{{ genre }}</RouterLink
                      >
                      <span v-if="index !== anime.genres.length - 1" class="mr-1">,</span>
                    </p>
                  </div>
                  <p class="text-sm mr-2">Premiered: {{ anime.season }} {{ anime.releaseDate }}</p>
                  <p class="text-sm mr-2">
                    Date Aired: {{ anime.startDate.year }}-{{ anime.startDate.month }}-{{
                      anime.startDate.day
                    }}
                  </p>
                  <p class="text-sm mr-2">
                    Date Ended: {{ anime.endDate.year }}-{{ anime.endDate.month }}-{{
                      anime.endDate.day
                    }}
                  </p>
                </div>
              </div>
            </div>
          </div>
        </div>

        <span class="text-lg">You may also like</span>
        <div
          class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 xl:grid-cols-6 gap-2 animated mt-5"
        >
          <RouterLink
            :to="'/anime/' + recommendation.id"
            v-for="recommendation in anime.recommendations.slice(0, 6)"
            :key="recommendation.id"
            class="card border border-dark-50 rounded-lg relative overflow-hidden group"
          >
            <div class="relative">
              <img
                :src="recommendation.image"
                :alt="recommendation.title.userPreferred"
                draggable="false"
                class="w-full h-60 md:h-80 object-cover transition-opacity duration-300"
              />
              <div
                class="absolute h-[102%] inset-0 bg-black opacity-0 group-hover:opacity-30 transition-opacity duration-300"
              ></div>
            </div>
            <div
              class="absolute inset-0 flex items-center justify-center opacity-0 group-hover:opacity-100 hover:animated hover:animated-fade-in"
            >
              <div i-carbon-play-filled class="text-white text-7xl" style="margin-top: -1em"></div>
            </div>
            <div class="p-4">
              <h3 class="font-semibold truncate">
                {{ recommendation.title?.userPreferred || recommendation.title?.english }}
              </h3>
              <p class="text-sm">Type: {{ recommendation.type }}</p>
              <div class="flex items-center">
                <div i-openmoji-star class="mr-1 py-2" />
                <span>{{ (recommendation.rating / 10).toFixed(1) }}</span>
              </div>
            </div>
          </RouterLink>
        </div>
      </div>
    </template>
    <template v-else>
      <V-InternalServer />
    </template>
  </div>
</template>

<script>
import axios from 'axios'
import videojs from 'video.js'
import 'video.js/dist/video-js.css'

import.meta.env.VITE_API_URL

export default {
  data() {
    return {
      isLoading: true,
      anime: null,
      selectedRange: '1-100',
      perPage: 100,
      provider: 'gogoanime',
      showFullDescription: false,
      selectedEpisode: null,
      selectedEpisodeUrl: '',
      activeEpisode: null,
      videoPlaying: false,
      showNotification: true
    }
  },
  computed: {
    filteredRelations() {
      return this.anime.relations.filter((relation) => this.isPrequelOrSequel(relation))
    },
    displayedEpisodes() {
      if (this.anime) {
        const [start, end] = this.selectedRange.split('-').map(Number)
        return this.anime.episodes.slice(start - 1, end)
      }
      return []
    },
    episodeRanges() {
      if (this.anime) {
        const totalEpisodes = this.anime.episodes.length
        const maxPerPage = this.perPage
        const rangeCount = Math.ceil(totalEpisodes / maxPerPage)
        if (rangeCount > 0) {
          return Array.from({ length: rangeCount }, (_, index) => {
            const start = index * maxPerPage + 1
            const end = Math.min((index + 1) * maxPerPage, totalEpisodes)
            return `${start}-${end}`
          })
        } else {
          return ['0']
        }
      }
      return []
    }
  },
  watch: {
    provider: {
      handler: 'updateProvider',
      immediate: true
    },
    '$route.params.id': {
      handler: 'updateProvider',
      immediate: true
    }
  },
  mounted() {
    document.addEventListener('keydown', this.handleKeyDown)
  },
  beforeUnmount() {
    document.removeEventListener('keydown', this.handleKeyDown)
  },
  methods: {
    findSelectedIndex() {
      return this.displayedEpisodes.findIndex(
        (episode) => episode.number === this.activeEpisode.number
      )
    },
    selectPreviousEpisode() {
      const selectedIndex = this.findSelectedIndex()
      if (selectedIndex > 0) {
        const prevEpisode = this.displayedEpisodes[selectedIndex - 1]
        this.selectEpisode(prevEpisode)
      }
    },
    selectNextEpisode() {
      const selectedIndex = this.findSelectedIndex()
      if (selectedIndex !== -1 && selectedIndex < this.displayedEpisodes.length - 1) {
        const nextEpisode = this.displayedEpisodes[selectedIndex + 1]
        this.selectEpisode(nextEpisode)
      }
    },
    stopVideoPlayer() {
      const video = document.getElementById('video-element')
      const player = videojs(video)
      if (!player.paused()) {
        player.load()
      }
    },
    isPrequelOrSequel(relation) {
      return relation.relationType === 'PREQUEL' || relation.relationType === 'SEQUEL'
    },
    handleKeyDown(event) {
      const video = this.$refs.videoElement
      if (event.keyCode === 37 && video) {
        video.currentTime -= 5
      } else if (event.keyCode === 39 && video) {
        video.currentTime += 5
      }
    },
    parseDescription(description) {
      const parser = new DOMParser()
      const doc = parser.parseFromString(description, 'text/html')
      return doc.body.innerHTML
    },
    async updateProvider() {
      this.isLoading = true
      const id = this.$route.params.id
      const provider = this.provider
      try {
        const response = await axios.get(
          `${import.meta.env.VITE_API_URL}/meta/anilist/info/${id}?provider=${provider}`
        )
        this.anime = response.data
        if (this.episodeRanges.length > 0) {
          this.selectedRange = this.episodeRanges[0]
          this.selectEpisode(this.displayedEpisodes[0])
        }
      } catch (error) {
        console.error(error)
      } finally {
        this.isLoading = false
      }
    },
    async fetchEpisodeUrl(episodeId) {
      try {
        let url
        if (this.provider === 'zoro') {
          const response = await axios.get(`${import.meta.env.VITE_API_URL}/anime/zoro${episodeId}`)
          url = response.data.sources[0].url
        } else if (this.provider === 'gogoanime') {
          const response = await axios.get(
            `${import.meta.env.VITE_API_URL}/anime/gogoanime${episodeId}`
          )
          url = response.data.sources[0].url
        }
        this.selectedEpisodeUrl = url
      } catch (error) {
        console.error(error)
      }
    },
    async selectEpisode(episode) {
      this.stopVideoPlayer()
      this.activeEpisode = episode
      try {
        const response = await axios.get(
          `${import.meta.env.VITE_API_URL}/anime/${this.provider}/watch/${episode.id.replace(
            /^\//,
            ''
          )}`
        )
        const sources = response.data.sources
        const defaultSource = sources.find((source) => source.quality === 'default')
        if (defaultSource) {
          this.selectedEpisodeUrl = defaultSource.url
          const video = document.getElementById('video-element')
          const player = videojs(video)
          player.src(this.selectedEpisodeUrl)
          player.play()
        }
      } catch (error) {
        console.error(error)
      }
    },
    closeNotification() {
      this.showNotification = false
    },
    goToRecommendation(id) {
      this.$router.push(`/anime/${id}`)
      window.scrollTo(0, 0)
    },
    created() {
      this.updateProvider()
      if (this.displayedEpisodes.length > 0) {
        this.selectedEpisode = this.displayedEpisodes[0]
        this.fetchEpisodeUrl(this.selectedEpisode.id)
      }
    }
  }
}
</script>

<style scoped>
.active-episode {
  background-color: rgba(34, 34, 34, var(--un-bg-opacity));
  color: rgba(254, 215, 170);
}

.truncate {
  max-height: 150px;
  overflow: hidden;
  position: relative;
}

.truncate::after {
  content: '';
  position: absolute;
  bottom: 0;
  right: 0;
  background: linear-gradient(to right, rgba(255, 255, 255, 0), white 80%);
  pointer-events: none;
}

.description-container {
  max-height: 250px;
  overflow-y: auto;
  position: relative;
}

.truncate {
  max-height: 150px;
  overflow: hidden;
  position: relative;
}

.truncate::after {
  content: '';
  position: absolute;
  bottom: 0;
  right: 0;
  background: linear-gradient(to right, rgba(255, 255, 255, 0), white 80%);
  pointer-events: none;
}

.description-container {
  max-height: 250px;
  overflow-y: auto;
  position: relative;
}
</style>
