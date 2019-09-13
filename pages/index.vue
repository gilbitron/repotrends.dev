<template>
  <div class="mx-auto" style="max-width: 1600px;">
    <div class="px-2 mb-2">
      <label class="inline-block rounded-full px-3 py-1 mr-1 cursor-pointer hover:bg-gray-200" v-for="lang in languages"
             :key="`lang-select-${lang}`">
        <input type="checkbox" class="mr-1" v-model="selectedLangs" :value="lang.toLowerCase()">
        <span class="text-gray-600">{{ lang }}</span>
      </label>
    </div>
    <div class="flex flex-wrap">
      <div class="text-gray-600 px-5" v-if="!hasData">
        Loading...
      </div>
      <div class="w-full md:w-1/2 lg:w-1/4" v-for="(repos, lang) in selectedData" :key="`lang-${lang}`">
        <div class="bg-white rounded-lg shadow-lg overflow-hidden m-4 mb-16">
          <div class="bg-gray-300 px-5 py-3">
            <h3 class="text-sm text-gray-600 font-bold tracking-wide uppercase">{{ lang }}</h3>
          </div>
          <ul>
            <li class="border-b px-5 py-3" v-for="repo in repos" :key="`repo-${repo.url}`">
              <div class="flex justify-between mb-2">
                <h4 class="truncate text-blue-500">
                  <a :href="repo.url" target="_blank" class="font-medium text-blue-500 hover:text-blue-700">
                    {{ nameFromUrl(repo.url) }}
                  </a>
                </h4>
                <span class="text-green-600 flex-shrink-0">
                  <svg class="inline-block fill-current w-3 h-3" style="margin-top: -5px;" viewBox="0 0 1792 1792"
                       xmlns="http://www.w3.org/2000/svg"><path
                    d="M1408 1216q0 26-19 45t-45 19h-896q-26 0-45-19t-19-45 19-45l448-448q19-19 45-19t45 19l448 448q19 19 19 45z"/></svg>
                  {{ repo.currentPeriodStars }}
                </span>
              </div>
              <p class="text-sm text-gray-600 leading-normal truncate mb-1">
                <span v-if="repo.description">{{ repo.description }}</span>
                <span class="text-gray-400" v-else>(no description)</span>
              </p>
              <p class="text-sm text-gray-600 leading-normal">
                <span class="leading-normal">
                  <svg class="inline-block fill-current w-3 h-3" style="margin-top: -4px;" viewBox="0 0 1792 1792"
                       xmlns="http://www.w3.org/2000/svg"><path
                    d="M1728 647q0 22-26 48l-363 354 86 500q1 7 1 20 0 21-10.5 35.5t-30.5 14.5q-19 0-40-12l-449-236-449 236q-22 12-40 12-21 0-31.5-14.5t-10.5-35.5q0-6 2-20l86-500-364-354q-25-27-25-48 0-37 56-46l502-73 225-455q19-41 49-41t49 41l225 455 502 73q56 9 56 46z"/></svg>
                  {{ formatNumber(repo.stars) }}
                </span>
              </p>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
    import _ from 'lodash';
    import {fetchRepositories} from '@huchenme/github-trending';

    export default {
        data() {
            return {
                data: {},
                selectedLangs: [],
            };
        },

        computed: {
            languages() {
                return ['JavaScript', 'Java', 'Python', 'PHP', 'C++', 'C#', 'TypeScript', 'Shell'];
            },
            languagesLower() {
                return _.map(this.languages, lang => lang.toLowerCase());
            },
            selectedData() {
                return _.pickBy(this.data, (val, key) => {
                    return _.includes(this.selectedLangs, key);
                });
            },
            hasData() {
                return !_.isEmpty(this.data);
            },
        },

        created() {
            this.selectedLangs = this.languagesLower;
        },

        mounted() {
            this.getData();
        },

        methods: {
            getData() {
                this.languagesLower.forEach(lang => {
                    fetchRepositories({language: lang, since: 'weekly'}).then(repos => {
                        this.$set(this.data, lang, _.reverse(_.sortBy(repos, repo => repo.currentPeriodStars)));
                    });
                });
            },
            nameFromUrl(repoUrl) {
                return repoUrl.replace('https://github.com/', '');
            },
            formatNumber(number) {
                return number.toLocaleString();
            },
        },
    };
</script>
