<template>
  <div>
    <v-dialog v-model="playDialog" v-if="playDialog">
      <video :src="playUrl" controls autoplay></video>
    </v-dialog>

    <v-list-item
      v-for="(clip, index) in pageClips"
      :key="clip.id"
      @click="playClip(clip, index)"
    >
      <v-img
        class="mr-2"
        :src="pageThumbnails[index]"
        contain
        max-height="100px"
        max-width="100px"
      >
      </v-img>
      <v-list-item-content class="">
        <v-list-item-title>
          {{ localTime(clip.startTime) }}
        </v-list-item-title>
        <v-list-item-subtitle>
          {{ clip.duration / 1000 }}s
        </v-list-item-subtitle>
      </v-list-item-content>
    </v-list-item>
    <div class="text-center" v-if="pages">
      <v-pagination v-model="page" :length="pages"></v-pagination>
    </div>
    <v-card-text v-else>No clips found.</v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn text small disabled>{{ pageRange }}</v-btn>
      <v-dialog width="unset" v-model="dialog">
        <template v-slot:activator="{ on }">
          <v-btn v-on="on" small>
            <v-icon x-small>fa fa-calendar-alt</v-icon>
            &nbsp;
            {{ year }}-{{ month }}-{{ date }}
          </v-btn>
        </template>
        <v-card>
          <v-date-picker @input="onDate"> </v-date-picker>
        </v-card>
      </v-dialog>
    </v-card-actions>
  </div>
</template>
<script>
import { datePickerLocalTimeToUTC } from "../common/date";
import { fetchClipThumbnail, fetchClipUrl } from "../common/videoclip";
import RPCInterface from "./RPCInterface.vue";
import Vue from "vue";

export default {
  mixins: [RPCInterface],

  asyncComputed: {
      pageRange: {
          async get() {
              const start = this.localTime(this.pageClips[0].startTime)
              const end = this.localTime(this.pageClips[this.pageClips.length - 1].startTime);
              return `${start} - ${end}`;
          }
      },
    pageClips: {
      async get() {
        const start = (this.page - 1) * 4;
        return this.clips.slice(start, start + 4);
      },
      default: [],
    },
    pageThumbnails: {
      async get() {
        const ret = Vue.observable([]);
        const clips = this.pageClips.slice();
        for (let i = 0; i < clips.length; i++) {
          ret.push("images/cameraloading.jpg");
        }
        (async () => {
          const mediaManager = this.$scrypted.mediaManager;
          for (let i = 0; i < clips.length; i++) {
            const clip = clips[i];
            if (!clip) {
              continue;
            }
            if (clip.id !== this.pageClips[i]?.id) {
              continue;
            }
            const thumbnail = await fetchClipThumbnail(
              mediaManager,
              this.rpc(),
              clip
            );
            Vue.set(ret, i, thumbnail);
          }
        })();
        return ret;
      },
      default: [],
    },
    clips: {
      async get() {
        const ret = await this.rpc().getVideoClips();
        return ret;
      },
      default: [],
    },
    pages: {
      async get() {
        return Math.ceil(this.clips.length / this.clipsPerPage);
      },
      default: 0,
    },
  },
  data() {
    return {
      playDialog: false,
      playUrl: null,
      clipsPerPage: 4,
      fetchingImage: null,
      fetchingImages: [],
      page: 1,
      dialog: false,
      date: new Date().getDate(),
      month: new Date().getMonth() + 1,
      year: new Date().getFullYear(),
    };
  },
  methods: {
    async playClip(clip) {
      const mediaManager = this.$scrypted.mediaManager;
      const url = await fetchClipUrl(mediaManager, this.rpc(), clip);
      this.playUrl = url;
      this.playDialog = true;
    },
    localTime(value) {
      const d = new Date(value);
      return d.toLocaleTimeString();
    },
    onDate(value) {
      this.page = 1;
      this.dialog = false;
      const dt = datePickerLocalTimeToUTC(value);
      const d = new Date(dt);
      this.month = d.getMonth() + 1;
      this.date = d.getDate();
      this.year = d.getFullYear();
    },
  },
};
</script>