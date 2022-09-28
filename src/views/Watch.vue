<template>
  <v-container id="Watch" class="fill-height" fluid>
    <div class="watchSection fill-height">
      <v-skeleton-loader large tile :loading="videoLoading">
        <v-responsive>
          <video
            controls
            style="
              height: 600px;
              width: 100%;
              background-color: black;
              border-radius: 5px;
            "
            @play="addVideoView"
          >
            <source :src="videoUrl !== null && videoUrl" type="video/mp4" />
          </video>
        </v-responsive>
      </v-skeleton-loader>

      <v-card class="videoDiv">
        <div class="videoInfoStyle">
          <div>
            <div>
              <h2 class="videoTitle">{{ watchVideo.title }}</h2>
              <h4>조회수 : {{ watchVideo.views }} 회</h4>
              <div
                @click="showDescription()"
                v-if="!isShowDescription"
                class="descriptionStyle"
              >
                설명: {{ moreShowDescription }}
              </div>
              <h5 v-else>설명: {{ watchVideo.description }}</h5>
            </div>
            <div class="videoLikeStyle">
              <h4>좋아요 {{ countVideoLike }}</h4>
              <h4>싫어요 {{ countVideoDislike }}</h4>
            </div>
          </div>
          <div class="likeBtnStyle">
            <v-btn v-on:click="addFeeling($event)">Like</v-btn>
            <v-btn v-on:click="addFeeling($event)">Dislike</v-btn>
          </div>
        </div>
        <div class="commentDiv">
          <h3>댓글 보기 {{ `(${countComment})` }}</h3>
          <div v-for="(comment, index) in comments" :key="index">
            <div class="commentStyle">
              <h4>{{ comment.userId.channelName }} :</h4>
              <div>{{ comment.text }}</div>
              <v-btn class="btnStyle" @click="loadTextField(index)">✏️</v-btn>
              <div></div>
              <v-btn class="btnStyle" @click="deleteComment(comment)">🗑️</v-btn>
              <div
                @click="showReply(index)"
                v-on:mouseenter="mouseEnterReply($event)"
                v-on:mouseleave="mouseleaveReply($event)"
                style="color: white"
              >
                답글
              </div>
              <div
                id="repliesId"
                v-show="isShowReplies[index]"
                class="replyStyle"
              >
                <div
                  v-for="(reply, j) in comment.replies"
                  :key="j"
                  style="display: flex"
                >
                  <h5>
                    {{ `${reply.userId.channelName} : ${reply.text} ` }}
                  </h5>
                  <v-btn
                    class="replyBtnStyle"
                    @click="updateReplies(reply, index)"
                    >답글 수정</v-btn
                  >
                  <v-btn class="replyBtnStyle" @click="deleteReplies(reply)"
                    >답글 삭제</v-btn
                  >
                </div>
              </div>
              <div class="newCommentStyle" v-show="isLoadText[index]">
                <v-textarea
                  block
                  v-model="newComment[index]"
                  placeholder="수정값 or 댓글 입력"
                ></v-textarea>
                <v-btn @click="updateComment(comment, index)">댓글 수정</v-btn>
                <v-btn @click="addReplies(comment, index)">답글 추가</v-btn>
              </div>
            </div>
          </div>
          <form @submit.prevent="addComment">
            <v-textarea
              v-model="writtenComment.text"
              placeholder="댓글 입력"
            ></v-textarea>
            <v-btn type="submit">댓글 추가</v-btn>
          </form>
        </div>
      </v-card>
    </div>

    <div class="listSection fill-height">
      <div class="listContainer">
        <VideoListCard
          v-for="video in videos"
          :key="video.id"
          :video="video"
          :channel="video.userId"
        ></VideoListCard>
      </div>
    </div>
  </v-container>
</template>
<script>
import axios from "axios";

import VideoListCard from "@/components/VideoListCard.vue";

export default {
  name: "Watch",
  data: () => ({
    videos: [],
    watchVideo: {},
    videoUrl: null,
    videoLoading: false,
    comments: [],
    writtenComment: {
      text: "",
      videoId: "",
      userId: "",
    },
    isShowReplies: [],
    newComment: [],
    isLoadText: [],
    isPlayVideo: false,
    isShowDescription: false,
  }),
  components: {
    VideoListCard,
  },
  watch: {
    $route(to, from) {
      if (to.path != from.path) {
        this.getWatchData(this.$route.params.id);
        this.getComments(this.$route.params.id);
      }
    },
    watchVideo() {
      this.videoUrl = `${process.env.VUE_APP_URL}/uploads/videos/${this.watchVideo.url}`;
    },
  },
  methods: {
    async getComments(videoId) {
      console.log("비디오 아이디: ", videoId);
      this.comments = [];
      await axios
        .get(process.env.VUE_APP_API + `/comments/${videoId}/videos`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log("영상 댓글 콘솔로그 : ", response);
          this.comments = response.data.data;
          this.comments.map((comment, index) => {
            if (this.isLoadText[index] === undefined) {
              this.isLoadText[index] = false;
            }
          });
          console.log("수정 텍스트 열렷냐? : ", this.isLoadText);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    async showReply(index) {
      this.isShowReplies[index] = !this.isShowReplies[index];
      console.log(this.isShowReplies[index]);
      this.isShowReplies.push();
    },
    async mouseEnterReply(event) {
      event.target.style.color = "skyblue";
    },
    async mouseleaveReply(event) {
      event.target.style.color = "white";
    },
    async loadTextField(index) {
      console.log("수정할 댓글 : ", index);
      for (let i = 0; i < this.isLoadText.length; i++) {
        console.log(i, this.isLoadText[index]);
        if (index != i) this.isLoadText[i] = false;
        else this.isLoadText[i] = !this.isLoadText[i];
      }
      this.isLoadText.push();
      // this.getComments(this.$route.params.id);
    },
    async getWatchData(id) {
      console.log(this.$route.params.id);
      this.videoLoading = true;
      this.watchVideo = {};
      await axios
        .get(process.env.VUE_APP_API + `/videos/${id}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log(
            "getWatchData - response : ",
            response,
            response.data.data
          );
          this.watchVideo = response.data.data;
          this.videoLoading = false;
        })
        .catch((error) => {
          console.log("getWatchData - error : ", error);
        });
    },
    async getVideos() {
      await axios
        .get(process.env.VUE_APP_API + "/videos/public", {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          this.isPlayVideo = false;
          console.log("getVideos - response : ", response, response.data.data);
          this.videos = response.data.data;
        })
        .catch((error) => {
          console.log("getVideos - error : ", error);
        });
    },
    async addComment() {
      if (this.writtenComment.text) {
        this.writtenComment.videoId = this.watchVideo.id;
        this.writtenComment.userId = this.watchVideo.userId;
        console.log(this.writtenComment.videoId);
        await axios
          .post(process.env.VUE_APP_API + "/comments", this.writtenComment, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          })
          .then((response) => {
            console.log("댓글 추가 : ", response);
            this.writtenComment.text = "";
            this.getComments(this.$route.params.id);
          })
          .catch((error) => {
            console.log(error);
          });
      } else alert("댓글을 입력해주세요!");
    },
    async deleteComment(comment) {
      console.log("삭제할 댓글: ", comment.id);
      await axios
        .delete(process.env.VUE_APP_API + `/comments/${comment.id}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          if (confirm("정말 삭제하시겠습니까?")) {
            console.log("삭제 성공! : ", response);
            console.log(comment);
            this.getComments(comment.videoId);
          }
        })
        .catch((error) => {
          if (comment.userId !== JSON.parse(localStorage.getItem("user")).id) {
            alert("권한이 없습니다. 니꺼만 지우세요.");
          }
          console.log(error);
        });
    },
    async updateComment(comment, index) {
      console.log(
        "수정할 댓글과 비디오 아이디: ",
        this.newComment[index],
        comment.id
      );
      // 수정할 값들을 댓글 수정창을 띄워서 입력하고 그 값을 보내주자.
      await axios
        .put(
          process.env.VUE_APP_API + `/comments/${comment.id}`,
          {
            text: this.newComment[index],
          },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((response) => {
          console.log("수정 성공 : ", response);
          this.newComment[index] = "";
          this.getComments(comment.videoId);
        })
        .catch((error) => {
          if (comment.userId !== JSON.parse(localStorage.getItem("user")).id) {
            alert("권한이 없습니다. 니꺼만 수정하세요.");
          }
          console.log("수정 실패 : ", error);
        });
    },
    async addReplies(comment, index) {
      console.log(comment, index);
      await axios
        .post(
          process.env.VUE_APP_API + "/replies",
          {
            text: this.newComment[index],
            commentId: comment.id,
            userId: comment.userId.id,
          },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((response) => {
          console.log("대댓글 추가 성공 : ", response);
          this.getComments(this.$route.params.id);
          this.newComment[index] = "";
        })
        .catch((error) => {
          console.log("대댓글 추가 실패", error);
        });
    },
    async updateReplies(reply, index) {
      console.log(reply);
      console.log("귀신이다", this.newComment[index]);
      await axios
        .put(
          process.env.VUE_APP_API + `/replies/${reply.id}`,
          {
            text: this.newComment[index],
          },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((response) => {
          console.log("답글 수정 성공 : ", response);
          this.newComment[index] = "";
          this.getComments(this.$route.params.id);
        })
        .catch((error) => {
          console.log("답글 수정 실패 : ", error);
        });
    },
    async deleteReplies(reply) {
      if (confirm("정말 삭제하시겠습니까?")) {
        console.log("삭제할 답글 : ", reply);
        await axios
          .delete(process.env.VUE_APP_API + `/replies/${reply.id}`, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          })
          .then((response) => {
            console.log("대댓글 삭제 성공 : ", response);
            this.getComments(this.$route.params.id);
          })
          .catch((error) => {
            if (reply.userId !== JSON.parse(localStorage.getItem("user")).id) {
              alert("권한이 없습니다. 니꺼만 지우세요.");
            }
            console.log("대댓글 삭제 실패 : ", error);
          });
      }
    },
    async addFeeling(event) {
      const text = event.target.innerText;
      console.log(text.toLowerCase());
      await axios
        .post(
          process.env.VUE_APP_API + "/feelings",
          {
            type: text.toLowerCase(),
            videoId: this.$route.params.id,
            userId: JSON.parse(localStorage.getItem("user")).id,
          },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((response) => {
          console.log("기분 추가 성공: ", response);
          this.getWatchData(this.$route.params.id);
        })
        .catch((error) => {
          console.log("기분 추가 실패 : ", error);
        });
    },
    async addVideoView() {
      console.log(this.isPlayVideo);
      if (!this.isPlayVideo) {
        await axios
          .put(
            process.env.VUE_APP_API + `/videos/${this.$route.params.id}/views`,
            {},
            {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            }
          )
          .then((response) => {
            console.log("비디오 조회수 증가 성공: ", response);
            this.isPlayVideo = true;
          })
          .catch((error) => {
            console.log("비디오 조회수 증가 실패 : ", error);
          });
      }
    },
    async showDescription() {
      if (
        this.watchVideo.description &&
        this.watchVideo.description.length > 5
      ) {
        this.isShowDescription = !this.isShowDescription;
      }
      console.log(this.isShowDescription);
    },
  },
  mounted() {
    this.getVideos();
    this.getWatchData(this.$route.params.id);
    this.getComments(this.$route.params.id);
  },
  computed: {
    moreShowDescription() {
      const description = this.watchVideo.description;
      if (description) {
        if (description.length > 5) {
          return `${description.substr(0, 5)}...더보기`;
        } else return description;
      } else return "";
    },
    countVideoLike() {
      return this.watchVideo.likes;
    },
    countVideoDislike() {
      return this.watchVideo.dislikes;
    },
    countComment() {
      return this.comments.length;
    },
  },
};
</script>
<style>
#Watch {
  background-color: rgb(246, 247, 247);
  display: grid;
  grid-template-columns: 70% 30%;
  padding: 2%;
}
.watchSection {
  display: grid;
  grid-gap: 20px;
}
.videoDiv {
  background-color: white;
  box-sizing: border-box;
  padding: 10px;
  width: 100%;
}
.videoInfoStyle {
  display: grid;
  grid-template-columns: 70% 30%;
}
.videoLikeStyle {
  display: grid;
  grid-template-columns: 15% 15%;
}
.likeBtnStyle {
  display: grid;
  grid-template-columns: 45% 45%;
  grid-gap: 10%;
}
.descriptionStyle {
  font-size: 13.28px;
  font-weight: bold;
}
.listSection {
  align-items: center;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding-left: 10px;
  width: 100%;
}
.listContainer {
  align-items: center;
  box-sizing: border-box;
  row-gap: 5px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  padding-left: 10px;
  width: 100%;
}
.commentDiv {
  border-radius: 5px;
  background-color: rgb(246, 247, 247);
  padding: 10px;
}
.commentStyle {
  display: grid;
  grid-template-columns: 20% 59% 10% 1% 10%;
  border-radius: 5px;
  background-color: rgb(206, 204, 204);
  padding: 10px;
  margin-bottom: 5px;
}
.btnStyle {
  background-color: rgb(46, 68, 63);
  padding: 5px;
}
.newCommentStyle {
  border-radius: 5px;
  display: grid;
  grid-template-columns: 70% 15% 15%;
  grid-column: 1 / span 5;
}
.newCommentStyle textarea {
  display: block;
}
.newCommentStyle button {
  justify-self: flex-end;
  align-self: flex-end;
}
.replyStyle {
  grid-column: 1 / span 2;
}
.replyBtnStyle {
  width: 10px;
}
</style>
