<script>
/* eslint-disable no-console */

import { required, helpers } from 'vuelidate/lib/validators';

import { storage, services, tracking } from '../modules';

import IconBase from '@/components/icons/IconBase';
import IconDevTo from '@/components/icons/IconDevTo';
import IconFacebook from '@/components/icons/IconFacebook';
import IconGitHub from '@/components/icons/IconGitHub';
import IconHackerNews from '@/components/icons/IconHackerNews';
import IconLinkedIn from '@/components/icons/IconLinkedIn';
import IconMedium from '@/components/icons/IconMedium';
import IconMicrosoft from '@/components/icons/IconMicrosoft';
import IconReddit from '@/components/icons/IconReddit';
import IconStackOverflow from '@/components/icons/IconStackOverflow';
import IconTwitter from '@/components/icons/IconTwitter';
import IconYouTube from '@/components/icons/IconYouTube';
import LinkCard from '@/components/LinkCard';

/* eslint-disable */
const customURL = helpers.regex(
  'customURL',
  /^(?:http(s)?:\/\/)?[\w.-]+(?:\.[\w\.-]+)+[\w\-\._~:/?#[\]@!\$&'\(\)\*\+,;=.]+$/
);

/* eslint-enable */

export default {
  name: 'LinkShare',
  components: {
    IconBase,
    IconTwitter,
    IconLinkedIn,
    IconReddit,
    IconFacebook,
    IconStackOverflow,
    IconHackerNews,
    IconMedium,
    IconDevTo,
    IconYouTube,
    IconGitHub,
    IconMicrosoft,
    LinkCard,
  },
  data() {
    return {
      copied: '',
      tactic: '',
      category: '',
      urlToShare: '',
      longLink: '',
      shortLink: '',
      shortenerProvider: '',
      shortApiKey: '',
      shortUsername: '',
      alias: '',
      showConfigurationError: false,
    };
  },
  validations: {
    urlToShare: {
      required,
      customURL,
    },
    tactic: {
      required,
    },
    category: {
      required,
    },
  },
  created() {
    this.getAlias();
  },
  methods: {
    async reloadSettings() {
      await Promise.all([
        this.getAlias(),
        this.getShortUsername(),
        this.getShortApiKey(),
        this.getShortenerProvider(),
      ]);
    },
    copySuccess() {
      this.$toasted.show('Copied to clipboard', {
        theme: 'outline',
        position: 'top-center',
        duration: 2000,
      });
    },
    /* eslint-disable */
    getAlias() {
      return storage.getters
        .alias()
        .then(result => (this.alias = result))
        .catch(err => console.error(err));
    },
    getShortUsername() {
      return storage.getters
        .shortUsername()
        .then(result => (this.shortUsername = result))
        .catch(err => console.error(err));
    },
    getShortApiKey() {
      return storage.getters
        .shortApiKey()
        .then(result => (this.shortApiKey = result))
        .catch(err => console.error(err));
    },
    getShortenerProvider() {
      return storage.getters
        .shortenerProvider()
        .then(result => (this.shortenerProvider = result));
    },
    /* eslint-enable */
    async create() {
      await this.reloadSettings();

      if (!this.alias || !this.shortenerProvider) {
        this.showConfigurationError = true;
        return;
      } else {
        this.showConfigurationError = false;
      }
      this.longLink = tracking.addTracking(
        this.urlToShare,
        this.tactic,
        this.category,
        this.alias
      );

      const short = { apiKey: this.shortApiKey, username: this.shortUsername };

      if (this.shortenerProvider && this.shortenerProvider === 'bit.ly') {
        services.bitly.shorten(this.longLink, short).then(response => {
          this.shortLink = response;
        });
      }
      if (this.shortenerProvider && this.shortenerProvider === 'cda.ms') {
        services.cda.shorten(this.longLink).then(response => {
          this.shortLink = response;
        });
      }
    },
    addTracking(tactic, category) {
      let ai = this.$appInsights;
      this.reloadSettings().then(() => {
        this.shortLink = '';
        this.tactic = tactic;
        this.category = category;
        this.longLink = tracking.addTracking(
          this.urlToShare,
          tactic,
          category,
          this.alias
        );
        ai.trackEvent({
          name: 'addTracking',
          properties: {
            tactic,
            category,
            alias: this.alias,
            url: this.urlToShare,
          },
        });
      });
    },
    twitter() {
      this.addTracking('twitter', 'social');
    },
    linkedin() {
      this.addTracking('linkedin', 'social');
    },
    reddit() {
      this.addTracking('reddit', 'social');
    },
    facebook() {
      this.addTracking('facebook', 'social');
    },
    stackoverflow() {
      this.addTracking('stackoverflow', 'social');
    },
    hackernews() {
      this.addTracking('hackernews', 'social');
    },
    azuremedium() {
      this.addTracking('azuremedium', 'blog');
    },
    medium() {
      this.addTracking('medium', 'blog');
    },
    youtube() {
      this.addTracking(this.tactic, 'youtube');
    },
    github() {
      this.addTracking(this.tactic, 'github');
    },
    devto() {
      this.addTracking('devto', 'blog');
    },
    microsoft() {
      this.addTracking('itopstalk', 'blog');
    },
  },
};
</script>

<template>
  <div class="wrapper">
    <h1>Share a Microsoft.com Link</h1>
    <v-card class="card">
      <v-card-title>
        <h3 class="grey--text">
          Tracking link format follows: tactic-category-alias
        </h3>
      </v-card-title>
      <v-flex md6 offset-md3>
        <v-alert type="warning" v-show="!alias">
          Beware, you have not yet set your Microsoft alias, needed to build
          this link. Go to
          <router-link to="/settings">Settings</router-link>&nbsp;to set your
          alias.
        </v-alert>
      </v-flex>
      <v-form>
        <v-container>
          <v-flex md6 offset-md3>
            <v-alert
              outlined
              :value="true"
              type="error"
              v-show="showConfigurationError"
            >
              Shortener Configuration Missing. Please go to
              <router-link to="/settings">Settings</router-link>&nbsp;to
              configure the missing settings
            </v-alert>
          </v-flex>
          <v-layout row wrap>
            <v-flex xs12>
              <v-text-field
                name="urlToShare"
                @blur="$v.urlToShare.$touch()"
                :class="{
                  'is-invalid': $v.urlToShare.$invalid && $v.urlToShare.$dirty,
                  'is-valid': !$v.urlToShare.$invalid,
                }"
                aria-describedby="urlToShare-describe"
                v-model="urlToShare"
                label="Url"
                prepend-icon="link"
              ></v-text-field>
            </v-flex>
            <v-flex xs5>
              <v-text-field
                id="tactic-code"
                name="tactic-code"
                @blur="$v.tactic.$touch()"
                :class="{
                  'is-invalid': $v.tactic.$invalid && $v.tactic.$dirty,
                  'is-valid': !$v.tactic.$invalid,
                }"
                aria-describedby="tactic-code-describe"
                v-model="tactic"
                label="Tactic"
                prepend-icon="tactic"
              ></v-text-field>
            </v-flex>
            <v-flex xs5>
              <v-text-field
                id="category-code"
                name="category-code"
                @blur="$v.category.$touch()"
                :class="{
                  'is-invalid': $v.category.$invalid && $v.category.$dirty,
                  'is-valid': !$v.category.$invalid,
                }"
                aria-describedby="category-code-describe"
                v-model="category"
                label="Category"
                prepend-icon="input"
              ></v-text-field>
            </v-flex>
            <v-flex md2>
              <v-btn
                :disabled="$v.$invalid"
                v-on:click="create"
                large
                color="primary"
                >Create Link</v-btn
              >
            </v-flex>
          </v-layout>
        </v-container>
      </v-form>
    </v-card>

    <LinkCard :longLink="longLink" :shortLink="shortLink" @copy="copySuccess" />

    <v-container>
      <v-layout row wrap>
        <v-flex md4>
          <v-card light class="card" min-height="350px">
            <v-card-title primary-title>
              <div>
                <div class="headline">Social Presets</div>
                <div class="preset-text">
                  category is set to <b>social</b> for the associated platform.
                </div>
                <div class="preset-example">
                  i.e. tactic-social-myalias
                </div>
              </div>
            </v-card-title>
            <v-container fluid grid-list-sm>
              <v-layout row wrap>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="twitter">
                    <icon-base icon-name="twitter">
                      <icon-twitter />
                    </icon-base>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="linkedin">
                    <icon-base icon-name="linkedin">
                      <icon-linked-in @click="linkedin" />
                    </icon-base>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="reddit">
                    <icon-base icon-name="reddit">
                      <icon-reddit @click="reddit" />
                    </icon-base>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="facebook">
                    <icon-base icon-name="facebook">
                      <icon-facebook @click="facebook" />
                    </icon-base>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="stackoverflow">
                    <icon-base icon-name="stackoverflow">
                      <icon-stack-overflow @click="stackoverflow" />
                    </icon-base>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="hackernews">
                    <icon-base icon-name="hackernews">
                      <icon-hacker-news @click="hackernews" />
                    </icon-base>
                  </v-btn>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card>
        </v-flex>
        <v-flex md4>
          <v-card light class="card" min-height="350px">
            <v-card-title primary-title>
              <div>
                <div class="headline">Blog Presets</div>
                <div class="preset-text">
                  category is set to
                  <b>blog</b> for the associated platform.
                </div>
                <div class="preset-example">
                  i.e. azuremedium-blog-myalias
                </div>
              </div>
            </v-card-title>
            <v-container fluid grid-list-sm>
              <v-layout row wrap>
                <v-flex xs6>
                  <v-btn large class="btn-icon" href="#" @click="azuremedium">
                    <span style="color:#326699">
                      <icon-base icon-name="azuremedium">
                        <icon-medium @click="azuremedium" />
                      </icon-base>
                    </span>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="medium">
                    <span style="color:#000">
                      <icon-base icon-name="medium">
                        <icon-medium @click="medium" />
                      </icon-base>
                    </span>
                  </v-btn>
                </v-flex>
                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="devto">
                    <icon-base icon-name="devto">
                      <icon-dev-to @click="devto" />
                    </icon-base>
                  </v-btn>
                </v-flex>

                <v-flex xs6>
                  <v-btn class="btn-icon" large href="#" @click="microsoft">
                    <icon-base icon-name="ITOpsTalk">
                      <icon-microsoft @click="microsoft" />
                    </icon-base>
                  </v-btn>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card>
        </v-flex>
        <v-flex md4>
          <v-card light class="card" min-height="350px">
            <v-card-title primary-title>
              <div>
                <div class="headline">Other Presets</div>
                <div class="preset-text">
                  Various quick links to set category. Uses the value from
                  <b>tactic</b>.
                </div>
              </div>
            </v-card-title>

            <v-container fluid grid-list-sm>
              <v-layout row wrap>
                <v-flex md6>
                  <v-btn class="btn-icon" large href="#" @click="youtube">
                    <icon-base icon-name="youtube">
                      <icon-you-tube @click="youtube" />
                    </icon-base>
                  </v-btn>
                </v-flex>
                <v-flex md6>
                  <v-btn class="btn-icon" large href="#" @click="github">
                    <icon-base icon-name="github">
                      <icon-git-hub @click="github" />
                    </icon-base>
                  </v-btn>
                </v-flex>
              </v-layout>
            </v-container>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<style scoped>
.link-card {
  margin: 5px;
}
.btn-icon {
  padding: 10px;
  margin: 10px;
}
.icon {
  margin: 10px;
}
.preset-text {
  font-size: 14px;
  word-break: break-word;
  line-height: 20px;
}
.preset-example {
  font-size: 14px;
  color: #1776d2;
  line-height: 20px;
}
</style>
