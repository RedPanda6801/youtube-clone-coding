<template>
  <div id="categoryDiv" class="grey lighten-5">
    <div class="categoryList">
      <v-card class="cardStyle" min-width="250" min-height="650">
        <v-list dense>
          <v-subheader class="categoryTitle">카테고리</v-subheader>
          <v-list-item-group
            v-model="selectedItem"
            color="gray"
            @change="getItem"
          >
            <v-list-item v-for="(title, index) in pagingCheck" :key="index">
              <!-- <v-list-item-icon>
              <v-icon v-text="title"></v-icon>
            </v-list-item-icon> -->
              <v-list-item-content>
                <v-list-item-title v-text="title"></v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>
        <div class="cardBottomStyle">
          <v-pagination
            id="pageStyle"
            v-model="page"
            class="pageStyle"
            :length="pageLength"
          ></v-pagination>
          <v-btn id="addBtn" class="btnStyle" @click="showAddDiv">✚</v-btn>
        </div>
      </v-card>
    </div>
    <v-card class="infoStyle">
      <div class="showInfoDiv">
        <h2>카테고리 정보</h2>
        <div class="descriptionStyle">
          <h4 style="background-color: white">
            ⌲ {{ selectedCategory.title }}
          </h4>
          <h5 style="background-color: white; height: 80px">
            ✎ {{ selectedCategory.description }}
          </h5>
        </div>
      </div>
      <form
        v-if="showUpdateDiv"
        id="formStyle"
        class="updateStyle"
        @submit.prevent="updateItem"
      >
        <h2>카테고리 수정</h2>
        <v-text-field
          class="inputStyle"
          v-model="updateForm.title"
          placeholder="최소 세글자를 입력해주세요"
        ></v-text-field>
        <v-textarea
          class="inputStyle"
          v-model="updateForm.description"
          placeholder="최소 세글자를 입력해주세요"
        ></v-textarea>
        <div class="btnDiv">
          <v-btn type="submit">수정</v-btn>
          <v-btn @click="deleteItem" disabled>삭제</v-btn>
        </div>
      </form>
      <form v-else id="addForm" class="addStyle" @submit.prevent="addItem">
        <h2>카테고리 추가</h2>
        <v-text-field
          class="inputStyle"
          v-model="updateForm.title"
          placeholder="최소 세글자를 입력해주세요"
        ></v-text-field>
        <v-textarea
          class="inputStyle"
          v-model="updateForm.description"
          placeholder="최소 세글자를 입력해주세요"
        ></v-textarea>
        <div class="btnDiv">
          <v-btn type="submit">추가</v-btn>
        </div>
      </form>
    </v-card>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "Category",
  data: () => ({
    categories: [],
    categoryTitles: [],
    updateForm: {
      title: "",
      description: "",
    },
    selectedCategory: {},
    selectedItem: 1,
    description: "",
    showUpdateDiv: true,
    updateBtn: "",
    page: 1,
  }),
  computed: {
    pagingCheck() {
      console.log("현재 페이지", this.page);
      const arr = [];
      this.categoryTitles.map((title, index) => {
        if (Math.floor(index / 12) + 1 == this.page) {
          console.log("13개까지만 보여줄게요 : ", title);
          arr.push(title);
        }
      });

      return arr;
    },
    pageLength() {
      return Math.ceil(this.categories.length / 12);
    },
  },
  methods: {
    async getCategories() {
      this.categoryLoading = true;
      await axios
        .get(process.env.VUE_APP_API + "/categories", {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          console.log("카테고리 불러오기 - response : ", response);
          this.categories = response.data.data;
          this.categoryTitles = response.data.data.map(
            (category) => category.title
          );
          console.log(
            "카테고리들 : ",
            this.categories,
            "타이틀: ",
            this.categoryTitles
          );
        })
        .catch((error) => {
          console.log("getCategories - error : ", error);
        })
        .finally(() => {
          this.categoryLoading = false;
        });
    },
    async getItem() {
      const item = this.selectedItem;
      this.showUpdateDiv = true;
      console.log("choosed item : ", item);
      this.categories.map((category, index) => {
        if (index === item) {
          this.selectedCategory = category;
          console.log(this.selectedCategory);
        }
      });
    },
    async updateItem() {
      console.log("submit");
      const categoryId = this.selectedCategory._id;
      console.log(categoryId);
      if (categoryId && this.updateForm) {
        await axios
          .put(
            process.env.VUE_APP_API + `/categories/${categoryId}`,
            this.updateForm,
            {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            }
          )
          .then((response) => {
            console.log("수정 성공 : ", response);
            this.getCategories();
          })
          .catch((error) => {
            console.log(error);
          });
      } else {
        alert("잘못된 입력 값입니다. 다시 입력해주세요.");
      }
    },
    async deleteItem() {
      const categoryId = this.selectedCategory._id;
      console.log(categoryId);
      if (categoryId) {
        if (confirm("정말 삭제하시겠습니까?")) {
          await axios
            .delete(process.env.VUE_APP_API + `/categories/${categoryId}`, {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            })
            .then((response) => {
              console.log("삭제 성공 : ", response);
              this.getCategories();
            })
            .catch((error) => {
              console.log(error);
            });
        }
      } else {
        console.log("didn't selected item! choose in list.");
      }
    },
    async showAddDiv() {
      this.showUpdateDiv = false;
    },
    async addItem() {
      if (this.updateForm.title && this.updateForm.description) {
        await axios
          .post(process.env.VUE_APP_API + "/categories", this.updateForm, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          })
          .then((response) => {
            console.log("추가 성공 : ", response);
            this.updateForm = {
              title: "",
              description: "",
            };
            this.getCategories();
          })
          .catch((error) => {
            console.log("추가 실패 : ", error);
          });
        return;
      } else {
        alert("입력되지 않은 값이 있습니다. 입력해주세요!");
        return;
      }
    },
  },
  mounted() {
    this.getCategories();
  },
};
</script>
<style>
#categoryDiv {
  margin: 3% 1% 1% 1%;
  display: grid;
  grid-template-columns: 25% 65%;
  grid-gap: 5%;
}
.categoryTitle {
  background-color: aliceblue;
}
.cardStyle {
  display: grid;
  grid-gap: 3%;
  grid-template-rows: calc(100% - 100px) 80px;
}
.infoStyle {
  display: grid;
  grid-gap: 3%;
  grid-template-rows: 30% 65%;
  background-color: white;
  padding: 3%;
}
.showInfoDiv {
  padding: 2%;
  background-color: rgb(246, 247, 247);
}
.descriptionStyle {
  display: grid;
  grid-gap: 10%;
}
.updateStyle {
  padding: 2%;
  background-color: rgb(246, 247, 247);
}
.addStyle {
  padding: 2%;
  background-color: rgb(246, 247, 247);
}
.inputStyle {
  font-size: 18px;
  background-color: white;
  padding: 2% 3% 0% 3%;
}
.btnStyle {
  width: 100%;
}
.btnDiv {
  padding: 5% 2% 1% 2%;
  display: grid;
  grid-template-columns: 20% 20%;
  grid-gap: 60%;
}
</style>
