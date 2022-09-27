<template>
  <div id="categoryDiv">
    <v-card class="categoryList" max-width="300" max-height="650" height="97%">
      <v-list dense>
        <v-subheader class="categoryTitle">카테고리 분류</v-subheader>
        <v-list-item-group
          v-model="selectedItem"
          color="primary"
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
        <v-pagination
          id="pageStyle"
          v-model="page"
          class="my-4"
          :length="pageLength"
        ></v-pagination>
      </v-list>
      <v-btn id="addBtn" class="btnStyle" @click="showAddDiv">✚</v-btn>
    </v-card>
    <div></div>
    <div class="updateDiv">
      <div class="showInfoDiv">
        <h1>카테고리 정보</h1>
        <h3>카테고리 이름 : {{ selectedCategory.title }}</h3>
        <h5>카테고리 설명 : {{ selectedCategory.description }}</h5>
      </div>
      <form
        v-if="showUpdateDiv"
        id="updateForm"
        class="formStyle"
        @submit.prevent="updateItem"
      >
        <h1>카테고리 수정</h1>
        <v-text-field
          v-model="updateForm.title"
          placeholder="title"
        ></v-text-field>
        <v-textarea
          class="textStyle"
          v-model="updateForm.description"
          placeholder="description"
        ></v-textarea>
        <div class="btnDiv">
          <v-btn type="submit">수정</v-btn>
          <div></div>
          <v-btn @click="deleteItem" disabled>삭제</v-btn>
        </div>
      </form>
      <form v-else id="addForm" class="formStyle" @submit.prevent="addItem">
        <h1>카테고리 추가</h1>
        <v-text-field
          v-model="updateForm.title"
          placeholder="title"
        ></v-text-field>
        <v-textarea
          class="textStyle"
          v-model="updateForm.description"
          placeholder="description"
        ></v-textarea>
        <v-btn type="submit">추가</v-btn>
      </form>
    </div>
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
      const arr = [];
      this.categoryTitles.map((title, index) => {
        if (Math.ceil(index / 12) == this.page || index / 12 === 0) {
          console.log(this.page);
          arr.push(title);
        }
      });
      console.log(this.categories.length);
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
            this.$router.go(0);
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
              this.$router.go(0);
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
            this.$router.go(0);
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
  display: grid;
  grid-template-columns: 25% 1% 72%;
  border-radius: 5px;
  height: 650px;
  background-color: lightgray;
}
.categoryList {
  display: grid;
  grid-template-rows: calc(100% - 35px) 35px;
  height: 650px;
  margin: 5px;
}
.categoryTitle {
  background-color: rgb(239, 237, 237);
}
.updateDiv {
  align-items: center;
  justify-content: center;
  margin: 5px 0px 5px 0px;
  border-radius: 10px;
  background-color: white;
  height: 97%;
  padding: 3%;
}
.formStyle {
  border-radius: 5px;
  border: 1px solid rgb(124, 119, 119);
  padding: 5%;
}
.btnStyle {
  bottom: 0;
}
.btnDiv {
  display: grid;
  grid-template-columns: 20% 60% 20%;
}
.textStyle {
  margin-bottom: 20px;
  border-radius: 5px;
  padding: 10px;
  border: 1px solid #000;
}
.pageStyle {
  width: 10px;
}
</style>
