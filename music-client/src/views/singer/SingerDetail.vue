<template>
  <el-container class="singer-detail">
    <el-aside class="album-slide">
      <div class="singer-img">
        <img :src="attachImageUrl(songDetails.pic)" alt="" />
      </div>
      <ul class="album-info">
        <li v-if="songDetails.sex !== 2">
          性别：{{ getUserSex(songDetails.sex) }}
        </li>
        <li>生日：{{ getBirth(songDetails.birth) }}</li>
        <li>故乡：{{ songDetails.location }}</li>
      </ul>
    </el-aside>
    <el-main class="album-main">
      <div class="intro">
        <h2>{{ songDetails.name }}</h2>
        <span>{{ songDetails.introduction }}</span>
      </div>
      <div class="album-body">
        <song-list :songList="currentSongList"></song-list>
      </div>
    </el-main>
  </el-container>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted } from "vue";
import { useStore } from "vuex";
import mixin from "@/mixins/mixin";
import SongList from "@/components/SongList.vue";
import { HttpManager } from "@/api";
import { getBirth } from "@/utils";

export default defineComponent({
  components: {
    SongList,
  },
  setup() {
    const store = useStore();
    const { getUserSex } = mixin();

    const currentSongList = ref([]);
    const songDetails = computed(() => store.getters.songDetails) as any;

    onMounted(async () => {
      try {
        const result = (await HttpManager.getSongOfSingerId(songDetails.value.id)) as any[];
        currentSongList.value = result;
      } catch (error) {
        console.error(error);
      }
    });

    return {
      songDetails,
      currentSongList,
      attachImageUrl: HttpManager.attachImageUrl,
      getBirth,
      getUserSex,
    };
  },
});
</script>

<style lang="scss" scoped>
@import "@/assets/css/singer-detail.scss";
</style>
