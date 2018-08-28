<template>
  <div class="ranking">

    <div v-if="isAfterGame && !isLoadingRankingUsers">
      <div v-if="isRankedIn">
        <div class="rank-post-done" v-if="rankPostDone">
          送信完了！
          <div class="rank-post-score">{{score | delimited}}点</div>
          <a class="twitter-share" :href="twitterShareUrl" target="_blank"><i class="fa fa-comment"></i>点数をツイート</a>
          <router-link to="/" class="button onemore-play">もう一度プレイ</router-link>
        </div>
        <div class="rank-post" v-else>
          <i class="fa fa-2x fa-spinner fa-pulse" v-if="isScoreSending"></i>
          <div v-else>
            <div class="rank-post-title">おめでとう！ハイスコアです</div>
            <div class="rank-post-score">{{score | delimited}}点</div>
            <div class="rank-post-nickname-title">ニックネーム</div>
            <input class="rank-post-nickname" type="text" v-model="nickname" placeholder="名無しさん">
            <div class="button rank-post-send" @click="postScore">送信</div>
          </div>
        </div>
      </div>
      <div class="rank-post" v-else>
        <div>
          <div class="rank-post-title">あなたのスコアは、</div>
          <div class="rank-post-score">{{score | delimited}}点</div>
          <a class="twitter-share" :href="twitterShareUrl" target="_blank"><i class="fa fa-comment"></i>点数をツイート</a>
          <router-link to="/" class="button onemore-play">もう一度プレイ</router-link>
        </div>
      </div>
    </div>

    <div class="ranking-user-list">
      <div class="ranking-user-list-title">ランキング</div>
      <i class="fa fa-2x fa-spinner fa-pulse" v-if="isLoadingRankingUsers"></i>
      <div class="ranking-user" v-for="(rankingUser, idx) in rankingUsers" :key="idx" :class="{'is-me': rankingUser.isMe}">
        <div class="ranking-user-rank">
          <i class="fa fa-crown" :class="'is-rank-' + (idx + 1)" v-if="idx <= 2"></i>
          <span>{{idx + 1}}</span>
        </div>
        <div class="ranking-user-name">{{rankingUser.nickname}}</div>
        <div class="ranking-user-score">{{rankingUser.score | delimited}}点</div>
      </div>
      <router-link to="/" class="button onemore-play">やさいぷよぷよ をプレイ</router-link>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Ranking',
  data() {
    return {
      apiUri: 'https://4i2mumy3r7.execute-api.ap-northeast-1.amazonaws.com/default/yasaipuyopuyo_score',
      rankingUsers: [],
      isScoreSending: false,
      rankPostDone: false,
      nickname: '',
      isLoadingRankingUsers: false,
    }
  },
  props: ['gameScore'],
  filters: {
    delimited(num) {
      return num.toLocaleString();
    },
  },
  mounted() {
    this.fetchRanking();
    window.ranking = this;
  },
  computed: {
    isRankedIn() {
      // ランカーが10人満たない場合は無条件にランクイン
      // ランキング最下位よりスコアが高く、過去に自分が今よりも高いスコアを取っていなければランクイン
      const rankingUsersLength = this.rankingUsers.length;
      return !this.rankingUsers.filter(o => o.userId === this.getUserId() && o.score >= this.score).length &&
        (
          rankingUsersLength < 10 ||
          this.rankingUsers[rankingUsersLength - 1].score < this.score
        );
    },
    isAfterGame() {
      return !!this.$store.state.score;
    },
    score() {
      return this.$store.state.score;
    },
    postParams() {
      let r = this.getParams;
      r.nickname = this.nickname || '名無しさん';
      r.score = this.score;
      return r;
    },
    getParams(user) {
      return {user_id: this.getUserId()}
    },
    twitterShareUrl() {
      const appUrl = 'http://yasai-puyopuyo.s3-website-ap-northeast-1.amazonaws.com/';
      const hashtags = 'やさいぷよぷよ';
      const message = `やさいぷよぷよで、${this.score}点とりました！`;
      return `https://twitter.com/share?url=${appUrl}&hashtags=${hashtags}&text=${message}`;
    },
  },
  methods: {
    setUserId(userId) {
      if(!window.sessionStorage) return null;
      return window.sessionStorage.setItem('yasaiPuyopuyoUserId', userId);
    },
    getUserId() {
      if(!window.sessionStorage) return null;
      const userId = window.sessionStorage.getItem('yasaiPuyopuyoUserId');
      return typeof userId === 'string' ? userId : null;
    },
    fetchRanking() {
      this.isLoadingRankingUsers = true;
      axios.get(this.apiUri, {params: this.getParams}).then((res)=>{
        this.rankingUsers = res.data.ranking;
        this.isLoadingRankingUsers = false;
      });
    },
    postScore() {
      this.isScoreSending = true;
      axios.get(this.apiUri, {params:this.postParams}).then((res)=>{
        this.isScoreSending = false;
        this.rankPostDone = true;
        this.rankingUsers = res.data.ranking;
        this.setUserId(res.data.userId);
      });
    },
  }
}
</script>

<style>
.ranking {
  max-width: 550px;
  margin: 0 auto 0;
  text-align: left;
}
.ranking-user-list {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  text-align: center;
}
.ranking-user-list-title {
  margin-bottom: 20px;
  font-weight: bold;
}
.ranking-user {
  text-align: left;
  display: flex;
  padding: 10px 0;
  border-bottom: 1px solid #efefef;
  align-items: center;
}
.ranking-user.is-me {
  background: #ffc;
}
.ranking-user-rank {
  font-weight: bold;
  font-size: 20px;
  width: 40px;
  font-family: Arial, sans-serif;
  padding-left: 1em;
  position: relative;
}
.ranking-user-score {
  font-family: Arial, sans-serif;
}
.ranking-user-name {
  flex: 1;
}
.ranking-user-rank i {
  font-size: 11px;
  position: absolute;
  left: 0;
  top: calc(50% - 0.5em);
  
}
.ranking-user-rank i.is-rank-1 { color: #bb9f67;}
.ranking-user-rank i.is-rank-2 { color: #878990;}
.ranking-user-rank i.is-rank-3 { color: #bf6b40;}
.rank-post {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
  text-align: center;
}
.rank-post-title {
  font-weight: bold;
}
.rank-post-score {
  font-size: 30px;
  font-family: Arial, sans-serif;
  font-weight: bold;
}
.rank-post-nickname-title {}
.rank-post-nickname {
  font-size: 20px;
  border: 1px solid #dadada;
  border-radius: 5px;
  width: 100%;
  padding: 5px;
  box-sizing: border-box;
  margin-bottom: 20px;
  background: #f9f9f9;
}

.button {
  display: block;
  background: #000;
  color: #fff;
  font-size: 14px;
  font-weight: bold;
  padding: 10px;
  border-radius: 5px;
  text-decoration: none;
}
.rank-post-send {
  background: #ffe000;
  color: #000;
}
.rank-post-done {
  border-radius: 10px;
  margin-bottom: 20px;
  background: #fff;
  padding: 20px;
  text-align: center;
}
.onemore-play {
  margin: 20px 20px 0;
}
</style>
