<template>
  <DefaultLayout>
    <div class="sidebar sticky overflow-hidden top-0 left-0 pt-16" :class="{ open: isSidebarOpen }" v-retain="isSidebarOpen">
      <AisInstantSearchSsr class="px-6 lg:px-16">
        <AisConfigure
          :hitsPerPage="hitsPerPage"
          :analyticsTags="['rotala']"
          :facetFilters="facetFilters"
        />

        <div class="my-6">
          <span class="flex justify-between">
            <AisSearchBox class="w-full" placeholder="Search extensions..." :classNames="{
              'ais-SearchBox-form': 'ro-input-group',
              'ais-SearchBox-input': 'ro-input ro-input-search',
              'ais-SearchBox-reset': 'hidden',
              'ais-SearchBox-submit': 'text-gray-600 absolute inset-y-0 left-0 pl-3 flex items-center'
            }">
              <template v-slot:submit-icon>
                <span class="ro-icon ro-icon-search"></span>
              </template>
            </AisSearchBox>
            <button class="button text-gray-700 text-lg flex items-center pr-0 md:hidden block" @click="toggleSidebar">
              <i class="ro-icon ro-icon-close icon-combo"></i>
            </button>
          </span>
          <AisStateResults class="my-4 ml-2" v-slot="{ nbHits }">
            <div class="text-xs text-gray-600">{{ nbHits }} results</div>
          </AisStateResults>
        </div>

        <hr class="hr" />

        <AisInfiniteHits
          :classNames="{
            'ais-InfiniteHits-list': 'list-none ml-0'
          }"
          class="overflow-y-scroll"
          style="height: calc(100vh - 11rem)">
          <template v-slot:item="{ item }">
            <li class="py-2 pl-4" :key="item.name">
              <g-link
                class="block relative bullet-dot"
                :class="{ active: $route.params.id == item.name }"
                :to="`/extensions/${item.name}`">
                <div class="flex justify-between">
                  <AisHighlight class="font-bold" :hit="item" attribute="name" />
                  <div class="flex w-16 text-xs text-gray-700 items-center ml-auto space-x-1">
                    <SVGIcon class="inline-block" width="12" height="12"  v-if="item.owner.name == 'daiyanze'" />
                    <span>{{ item.humanDownloadsLast30Days }}</span>
                    <i style="font-size:10px;" class="ro-icon ro-icon-arrow-down"></i>
                  </div>
                </div>
                <AisHighlight class="block text-sm text-gray-700 py-4" :hit="item" attribute="description" />
              </g-link>
            </li>
          </template>

          <template v-slot:loadMore="{ refine, isLastPage }">
            <div class="px-4 mb-16 pb-16">
              <button class="ro-button ro-button-primary w-full" @click="refine" :disabled="isLastPage">
                More +
              </button>
            </div>
          </template>
        </AisInfiniteHits>
      </AisInstantSearchSsr>
    </div>

    <div class="content p-6">
      <button class="ro-button text-gray-700 text-lg flex items-center px-0 mb-8 md:hidden block" @click="toggleSidebar">
        <i class="ro-icon ro-icon-chevron-left ro-icon-combo"></i> Extensions
      </button>

      <div class="flex text-gray-700 text-sm my-2" v-if="Boolean($route.params.id) && hit">
        <a
          rel="noopener noreferrer"
          target="_blank"
          :href="hit.repository.url"
          :title="`View on ${hit.repository.host}`"
          :aria-label="`View on ${hit.repository.host}`"
          class="link inline-flex items-center mr-6">
          <div :is="repositoryIcon(hit.repository)" class="inline-block" width="16" height="16" />&nbsp;
          View on {{ hit.repository.host }}
        </a>

        <a
          rel="noopener noreferrer"
          target="_blank"
          v-if="hit.repository.project == 'rotala'" href="https://www.npmjs.com/org/rotala"
          title="View Rotala on NPM"
          aria-label="View Rotala on NPM"
          class="link inline-flex items-center">
          <SVGIcon width="16" height="16" />&nbsp;
          Official
        </a>
      </div>

      <div class="markdown" v-html="content" v-if="Boolean($route.params.id) && hit"></div>
      <div class="welcome-block text-center mx-auto max-w-2xl px-6 mt-8" v-else>
        <SVGExtension class="mb-12 mx-auto" width="300" />
        <h3>Welcome to the Rotala Extensions Library!</h3>
        <p class="mt-12">Use the search box to pick an extension.
          With Rotala extensions, you can quickly extend the default behaviors and appearances of Rotala components to make everything look better.</p>
        <p class="mt-8">Learn how to build your own <g-link class="ro-link ro-link-doc" to="/docs/extension">extension</g-link></p>
      </div>
    </div>
  </DefaultLayout>
</template>

<script>
import markdown from '~/utils/markdown'
import Sidebar from '~/components/Sidebar.vue'
import algoliasearch from 'algoliasearch/lite'
import SVGGithub from '~/assets/img/github.svg'
import SVGGitLab from '~/assets/img/gitlab.svg'
import SVGBitbucket from '~/assets/img/bitbucket.svg'
import SVGExtension from '~/assets/img/extension.svg'
import SVGIcon from '~/assets/img/icon-fill-current.svg'

import {
  createInstantSearch,
  AisInstantSearchSsr,
  AisStateResults,
  AisInfiniteHits,
  AisHighlight,
  AisConfigure,
  AisSearchBox
} from 'vue-instantsearch'

const searchClient = algoliasearch(
  process.env.GRIDSOME_ALGOLIA_NPM_APP_ID,
  process.env.GRIDSOME_ALGOLIA_NPM_SEARCH_API_KEY
)

const { instantsearch, rootMixin } = createInstantSearch({
  indexName: 'npm-search',
  searchClient
})

export default {
  mixins: [
    rootMixin
  ],
  components: {
    Sidebar,
    AisSearchBox,
    AisConfigure,
    AisHighlight,
    AisInfiniteHits,
    AisStateResults,
    AisInstantSearchSsr,
    SVGGithub,
    SVGGitLab,
    SVGBitbucket,
    SVGExtension,
    SVGIcon
  },
  data () {
    return {
      hit: null,
      hitsPerPage: 50,
      facetFilters: [
        'keywords:rotala-extension',
        'deprecated:false'
      ],
      isSidebarOpen: false
    }
  },
  computed: {
    content () {
      return this.hit && this.hit.readme
        ? markdown(this.hit.readme)
        : ''
    }
  },
  serverPrefetch () {
    return instantsearch.findResultsState({
      hitsPerPage: this.hitsPerPage,
      facetFilters: this.facetFilters
    })
  },
  metaInfo () {
    const meta = {
      title: 'Extensions',
      meta: []
    }

    if (this.hit) {
      meta.title = this.hit.name

      if (this.hit.description) {
        meta.meta.push({
          key: 'description',
          name: 'description',
          content: this.hit.description
        })
      }
    }

    return meta
  },
  methods: {
    async fetchCurrent () {
      if (!Boolean(this.$route.params.id)) {
        return
      }

      const { id: name } = this.$route.params
      const { results: [results] } = await searchClient.search([{
        indexName: 'npm-search',
        query: name
      }])

      this.hit = results.hits.find(hit => hit.name === name)
      this.isSidebarOpen = false
    },
    repositoryIcon (repository) {
      switch (repository.host) {
        case 'github.com': SVGGithub; break
        case 'gitlab.com': SVGGitLab; break
        case 'bitbucket.com': SVGBitbucket; break
      }

      return SVGGithub
    },
    toggleSidebar () {
      this.isSidebarOpen = !this.isSidebarOpen
    }
  },
  watch: {
    $route: {
      handler: 'fetchCurrent',
      immediate: true
    }
  }
}
</script>

<style scoped>
.sidebar {
  overflow-y: hidden !important;
  max-width: calc(28rem);
  transform: translateX(calc(-28rem));
  transition: transform .2s;
}

.sidebar.open {
  transform: translate(0);
}

.sidebar.open + .content {
  transform: translate(calc(28rem));
}

.content {
  margin-top: calc(-100vh + 4rem - 4px);
  transition: transform .2s;
  max-width: 960px;
}

@media screen and (min-width: 768px) {
  .content {
    width: calc(100% - 28rem);
    transform: translate(calc(28rem));
  }

  .sidebar {
    transform: translateX(0);
  }
}

.welcome-block {
  height: calc(100vh - 9rem);
}

.markdown {
 min-height: calc(90vh);
 padding-bottom: calc(100vh - 13rem + 2px);
}
</style>