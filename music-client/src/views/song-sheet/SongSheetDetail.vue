<template>
  <el-container class="song-sheet-detail">
    <el-aside class="album-slide">
      <div class="album-img">
        <img :src="attachImageUrl(songDetails.pic)" alt="" />
      </div>
      <div class="album-info">
        <h2>简介：</h2>
        <span> {{ songDetails.introduction }} </span>
      </div>
    </el-aside>
    <el-main class="album-main">
      <div class="album-title">
        <p>{{ songDetails.title }}</p>
      </div>
      <!--评分-->
      <div class="album-score">
        <div>
          <h3>歌单评分</h3>
          <el-rate v-model="rank" allow-half disabled></el-rate>
        </div>
        <span>{{ rank * 2 }}</span>
        <div>
          <h3>{{ assistText }}</h3>
          <el-rate allow-half v-model="score" :disabled="disabledRank" @click="pushValue()"></el-rate>
        </div>
      </div>
      <!--歌曲-->
      <div class="album-body">
        <song-list :songList="currentSongList"></song-list>
        <comment :playId="songListId" :type="1"></comment>
      </div>
    </el-main>
  </el-container>
</template>

<script lang="ts">
import { defineComponent, ref, computed, getCurrentInstance } from "vue";
import { useStore } from "vuex";
import mixin from "@/mixins/mixin";
import SongList from "@/components/SongList.vue";
import Comment from "@/components/Comment.vue";
import { HttpManager } from "@/api";

export default defineComponent({
  components: {
    SongList,
    Comment,
  },
  setup() {
    const { proxy } = getCurrentInstance();
    const store = useStore();
    const { checkStatus } = mixin();

    const currentSongList = ref([]); // 存放的音乐
    const songListId = ref(""); // 歌单 ID
    const score = ref(0);
    const rank = ref(0);
    const disabledRank = ref(false);
    const assistText = ref("评价");
    // const evaluateList = ref(["很差", "较差", "还行", "推荐", "力推"]);
    const songDetails = computed(() => store.getters.songDetails); // 单个歌单信息
    const userId = computed(() => store.getters.userId);

    songListId.value = songDetails.value.id; // 给歌单ID赋值

    // 收集歌单里面的歌曲
    async function getSongId(id) {
      const result = (await HttpManager.getListSongOfSongId(id)) as any[];
      // 获取歌单里的歌曲信息
      for (const item of result) {
        // 获取单里的歌曲
        const resultSong = await HttpManager.getSongOfId(item.songId);
        currentSongList.value.push(resultSong[0]);
      }
    }
    // 获取评分
    async function getRank(id) {
      const result = (await HttpManager.getRankOfSongListId(id)) as number;
      rank.value = result / 2;
    }
    async function getUserRank(userId, songListId) {
      const result = (await HttpManager.getUserRank(userId, songListId)) as string;
      score.value = parseInt(result);
      disabledRank.value = true;
      assistText.value = "已评价";
    }
    // 提交评分
    async function pushValue() {
      if (disabledRank.value || !checkStatus()) return;

      const params = new URLSearchParams();
      params.append("songListId", songListId.value);
      params.append("consumerId", userId.value);
      params.append("score", (score.value * 2).toString());

      try {
        const result = (await HttpManager.setRank(params)) as ResponseBody;
        (proxy as any).$message({
          message: result.message,
          type: result.type,
        });

        if (result.success) {
          getRank(songListId.value);
          disabledRank.value = true;
        }
      } catch (error) {
        console.error(error);
      }
    }

    getUserRank(userId.value, songListId.value);
    getRank(songListId.value); // 获取评分
    getSongId(songListId.value); // 获取歌单里面的歌曲ID

    return {
      songDetails,
      rank,
      score,
      disabledRank,
      assistText,
      currentSongList,
      songListId,
      attachImageUrl: HttpManager.attachImageUrl,
      pushValue,
    };
  },
});
</script>

<style lang="scss" scoped>
@import "@/assets/css/song-sheet-detail.scss";
</style>
