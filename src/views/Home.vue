<template>
  <div class="app">
    <my-header></my-header>
    <div class="main-container">
      <my-sidebar
        @onChangeTag="onChangeTag"
        @onTagsLoaded="onTagsInit"
      ></my-sidebar>
      <div class="content">
        <v-container>
          <!--这里显示管理员个人卡片-->
          <profile-card></profile-card>

          <!--这里显示管理员说明文档-->
          <div v-if="ifUsageShowed" class="card">
            <v-card-title>
              <div class="content-title">说明</div>
            </v-card-title>
            <v-card-text class="content-text">
              本平台分为两办管理员和子管理员。两办管理员可以管理学生提出问题的“其他”标签和子管理员退回的标签。
              同时可以在问题下添加标签。子管理员可以管理自己标签下的问题并进行回复，如果觉得不属于本标签的问题，可以退回问题。
            </v-card-text>
          </div>

          <div v-else id="question-block">
            <!--这里显示问题-->
            <div v-if="permission === 'false'" class="expansion-box">
              <!--扩展面板-->

              <div
                v-for="(item, i) in currentQuestions"
                :key="i"
                class="each-question-box card"
              >
                <div class="question-title">
                  {{ item.name }}
                </div>

                <div class="datetime-style">
                  {{ formatTime(item.updated_at) }}
                </div>

                <!--问题描述-->
                <div class="each-question-description">描述</div>
                <v-card-text class="each-question-content">
                  {{ item.description }}
                  <!--问题图片-->
                  <image-grid :question-id="item.id"></image-grid>
                </v-card-text>

                <v-tabs center-active grow v-model="controlTab">
                  <v-tab v-for="item in controlTabItems" :key="item.tab">{{
                    item.title
                  }}</v-tab>
                </v-tabs>

                <v-tabs-items v-model="controlTab">
                  <v-tab-item key="updateTag">
                    <v-chip style="margin: 7px 0"
                      >流转原因:
                      {{
                        item.admin_commit ? item.admin_commit : "未知"
                      }}</v-chip
                    >
                    <div
                      v-for="tag in item.tags"
                      :key="tag.id"
                      class="item-tags"
                    >
                      <div v-text="tag.name"></div>
                    </div>

                    <!--退回操作-->
                    <div>
                      <return-back-text-editor
                        :questionId="item.id"
                        v-on:getReason="getReason"
                      ></return-back-text-editor>
                    </div>
                  </v-tab-item>

                  <!--这里要加上其他管理员的评论-->
                  <v-tab-item key="addComment">
                    <v-card flat>
                      <div v-if="!item.solved" style="margin: 7px 0">
                        <span class="btn-font-style green"> 未解决 </span>
                      </div>

                      <div v-else style="margin: 7px 0">
                        <span class="btn-font-style red"> 已解决 </span>
                      </div>

                      <!--管理员回复图标-->
                      <!--学生评论图标-->
                      <div class="admin-student-icon">
                        <admin-answer :current-question="item"></admin-answer>
                        <student-comment
                          :current-question="item"
                        ></student-comment>
                      </div>

                      <!--富文本编辑器-->
                      <div style="margin-top: 25px">
                        <my-quill-editor
                          :question-id="Number(item.id)"
                          v-on:getAnswerByChild="getAnswerByChild"
                        >
                        </my-quill-editor>
                      </div>
                    </v-card>
                  </v-tab-item>
                </v-tabs-items>
              </div>

              <div class="pagination-box">
                <v-pagination
                  color="#1e88e5"
                  v-model="page_1"
                  :length="childPage"
                ></v-pagination>
              </div>
            </div>

            <!--这里是两办管理员-->
            <div v-else-if="permission === 'true'" style="margin-top: 15px">
              <div class="expansion-box">
                <div
                  v-for="(item, i) in currentQuestions"
                  :key="i"
                  class="each-question-box card"
                >
                  <div class="question-title">{{ item.name }}</div>
                  <div class="datetime-style">
                    {{ formatTime(item.updated_at) }}
                  </div>

                  <div class="each-question-description">描述</div>
                  <v-card-text class="each-question-content">
                    {{ item.description }}
                    <!--问题图片-->
                    <image-grid :question-id="item.id"></image-grid>
                  </v-card-text>

                  <v-tabs center-active grow v-model="controlTab">
                    <v-tab v-for="item in controlTabItems" :key="item.tab">{{
                      item.title
                    }}</v-tab>
                  </v-tabs>
                  <v-tabs-items v-model="controlTab" style="margin-top:7px">
                    <v-tab-item key="updateTag">
                      <v-chip
                        >流转原因：
                        {{
                          item.admin_commit ? item.admin_commit : "未知"
                        }}</v-chip
                      >

                      <v-list-item v-for="tag in item.tags" :key="tag.id">
                        <v-list-item-content>
                          <v-list-item-title
                            v-text="tag.name"
                          ></v-list-item-title>
                        </v-list-item-content>

                        <v-list-item-action>
                          <v-btn icon @click="deleteTag(item.id, tag.id)">
                            <v-icon color="grey lighten-1">mdi-delete</v-icon>
                          </v-btn>
                        </v-list-item-action>
                      </v-list-item>

                      <!--流转原因封装成组件-->
                      <div>
                        <tag-search-column
                          :current-question="item"
                        ></tag-search-column>
                      </div>
                    </v-tab-item>

                    <v-tab-item key="addComment">
                      <v-card flat>
                        <div v-if="!item.solved" style="margin: 7px 0">
                          <span class="btn-font-style green"> 未解决 </span>

                          <!--管理员回复图标-->
                          <!--学生评论图标-->
                          <div class="admin-student-icon">
                            <admin-answer
                              :current-question="item"
                            ></admin-answer>
                            <student-comment
                              :current-question="item"
                            ></student-comment>
                          </div>
                        </div>

                        <div v-else>
                          <span class="btn-font-style red"> 已解决 </span>

                          <!--管理员回复图标-->
                          <!--学生评论图标-->

                          <div class="admin-student-icon">
                            <admin-answer
                              :current-question="item"
                            ></admin-answer>
                            <student-comment
                              :current-question="item"
                            ></student-comment>
                          </div>
                        </div>
                      </v-card>
                    </v-tab-item>
                  </v-tabs-items>
                </div>

                <div class="pagination-box">
                  <v-pagination
                    color="#1e88e5"
                    v-model="page_2"
                    :length="LBPage"
                  ></v-pagination>
                </div>
              </div>
            </div>
          </div>
        </v-container>
      </div>
    </div>
  </div>
</template>

<script>
import MyHeader from "../components/Header";
import MySidebar from "../components/Sidebar";
import ProfileCard from "../components/ProfileCard";
import TagSearchColumn from "../components/TagSearchColumn";
import AdminAnswer from "../components/AdminAnswer";
import StudentComment from "../components/StudentComment";
import MyQuillEditor from "../components/tools/MyQuillEditor";
import ImageGrid from "../components/tools/ImageGrid";
import returnBackTextEditor from "@/components/tools/returnBackTextEditor";
import {
  addComment,
  addQuestionTag,
  getAnswerByQuestion,
  getQuestionsByTag,
  getTagByQuestion,
  removeTagByQuestion,
  getCommitByQuestion,
} from "../api/admin";
import { getUser, setTagList } from "../utils/cookie";

const toolbarOptions = [
  ["bold", "italic", "underline", "strike"], // toggled buttons

  [{ header: 1 }, { header: 2 }], // custom button values

  [{ size: ["small", false, "large", "huge"] }], // custom dropdown
  [{ header: [1, 2, 3, 4, 5, 6, false] }],

  [{ color: [] }, { background: [] }], // dropdown with defaults from theme
  [{ align: [] }],
  ["clean"],
];
export default {
  name: "Home",
  data: function () {
    return {
      isShowComment: false,
      permission: getUser().isLB,
      tagsList: [],
      currentTagId: 0,
      ifUsageShowed: true,
      currentQuestions: [],

      editorOption: {
        placeholder: "请输入评论",
        theme: "snow",
        modules: {
          toolbar: {
            container: toolbarOptions,
          },
        },
      },
      controlTab: null,
      controlTabItems: [
        { tab: "updateTag", title: "修改标签" },
        { tab: "addComment", title: "添加回复" },
      ],

      //分页
      page_1: 1,
      page_2: 1,

      childPage: 0,
      LBPage: 0,
    };
  },

  components: {
    TagSearchColumn,
    MyHeader,
    MySidebar,
    ProfileCard,
    StudentComment,
    AdminAnswer,
    MyQuillEditor,
    ImageGrid,
    returnBackTextEditor,
  },

  watch: {
    page_1: function (page) {
      const data = {
        id: getUser().id,
        token: getUser().token,
        tag_id: this.currentTagId,
        limits: 10,
        page: page,
      };

      //获取当前问题列表
      getQuestionsByTag(data)
        .then((res) => {
          //console.log(res);
          const response = res.data;
          if (response.ErrorCode === 1) {
            alert("拉取问题失败:" + res.data.msg);
          } else if (response.ErrorCode === 0) {
            //对tags排序
            response.data.data.forEach((quesItem) => {
              quesItem.tags.sort((a, b) => {
                return a.id - b.id;
              });
            });
            this.currentQuestions = response.data.data;
            //console.log(this.currentQuestions);

            //获取当前问题的管理员的回复
            //获取当前问题学生的评论

            for (let i = 0; i < this.currentQuestions.length; i++) {
              const id = this.currentQuestions[i].id;
              this.getAnswerByQuestion(id, i);
              this.getCommentByQuestion(id, i);
            }
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },

    page_2: function (page) {
      const data = {
        id: getUser().id,
        token: getUser().token,
        tag_id: this.currentTagId,
        limits: 10,
        page: page,
      };

      //获取当前问题列表
      getQuestionsByTag(data)
        .then((res) => {
          //console.log(res);
          const response = res.data;
          if (response.ErrorCode === 1) {
            alert("拉取问题失败:" + res.data.msg);
          } else if (response.ErrorCode === 0) {
            //对tags排序
            response.data.data.forEach((quesItem) => {
              quesItem.tags.sort((a, b) => {
                return a.id - b.id;
              });
            });
            this.currentQuestions = response.data.data;
            //console.log(this.currentQuestions);

            //获取当前问题的管理员的回复
            //获取当前问题学生的评论

            for (let i = 0; i < this.currentQuestions.length; i++) {
              const id = this.currentQuestions[i].id;
              this.getAnswerByQuestion(id, i);
              this.getCommentByQuestion(id, i);
            }
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },

  methods: {
    formatTime: function (updated_at) {
      let now = new Date();
      let target_year = updated_at.substr(0, 4);
      let target_month = updated_at.substr(5, 2);
      let target_day = updated_at.substr(8, 2);
      let target_hour = (updated_at.substr(11, 2) - 0 + 8) % 24; //后端bug相差8小时
      let target_minute = updated_at.substr(14, 2);
      let out_year = "",
        out_month = "",
        out_day = "",
        out_hour = target_hour,
        out_minute = target_minute;
      switch (now.getFullYear() - target_year) {
        case 0:
          out_year = "";
          break;
        case 1:
          out_year = "去年";
          break;
        case 2:
          out_year = "前年";
          break;
        default:
          out_year = target_year + "年";
          break;
      }
      if (now.getMonth() + 1 - target_month == 0) {
        if (now.getDate() - target_day < 3) {
          switch (now.getDate() - target_day) {
            case 0:
              out_day = "今天";
              break;
            case 1:
              out_day = "昨天";
              break;
            case 2:
              out_day = "前天";
              break;
          }
        } else {
          out_month = target_month - 0 + "月";
          out_day = target_day - 0 + "日";
        }
      } else {
        out_month = target_month - 0 + "月";
        out_day = target_day - 0 + "日";
      }

      return out_year + out_month + out_day + " " + out_hour + ":" + out_minute;
    },
    onChangeTag: function (tagId) {
      if (tagId.length === 0 || tagId[0] === 0) {
        this.ifUsageShowed = true;
      } else {
        this.ifUsageShowed = false;
        this.currentTagId = tagId[0];

        const data = {
          id: getUser().id,
          token: getUser().token,
          tag_id: tagId[0],
          limits: 10,
          page: 1,
        };

        //获取当前问题列表
        getQuestionsByTag(data)
          .then((res) => {
            //console.log(res);
            const response = res.data;
            if (response.ErrorCode === 1) {
              alert("拉取问题失败:" + res.data.msg);
            } else if (response.ErrorCode === 0) {
              //对tags排序
              response.data.data.forEach((quesItem) => {
                quesItem.tags.sort((a, b) => {
                  return a.id - b.id;
                });
              });
              this.currentQuestions = response.data.data;
              //console.log(this.currentQuestions);
              //this.childPage = response.total;
              if (this.permission === "false") {
                this.childPage = Math.ceil(response.data.total / 10);
              } else {
                this.LBPage = Math.ceil(response.data.total / 10);
              }

              //获取当前问题的管理员的回复
              //获取当前问题学生的评论

              for (let i = 0; i < this.currentQuestions.length; i++) {
                const id = this.currentQuestions[i].id;
                this.getAnswerByQuestion(id, i);
                this.getCommentByQuestion(id, i);
              }
            }
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },

    //删除问题标签
    deleteTag(questionId, tagId) {
      const data = {
        id: Number(getUser().id),
        token: getUser().token,
        question_id: questionId,
        tagList: JSON.stringify([tagId]),
      };

      removeTagByQuestion(data).then((res) => {
        const response = res.data;
        if (response.ErrorCode === 1) {
          alert("删除失败" + response.msg);
          console.log(data);
        } else {
          alert("删除成功");
        }
      });
    },

    onAddTag(questionId, select, reason) {
      if (typeof select === "undefined") {
        alert("未选择所要添加的标签");
      } else if (reason === "undefined") {
        alert("未填写流转原因");
      } else {
        const data = {
          id: getUser().id,
          token: getUser().token,
          question_id: questionId,
          tagList: JSON.stringify([select[3]]), //这里或许要转换一下格式,
          reason: reason,
        };

        //增加标签
        addQuestionTag(data).then((res) => {
          const response = res.data;
          if (response.ErrorCode === 1) {
            alert("添加标签失败");
          } else {
            const otherTag = this.searchTagId("其他");
            const removeData = {
              id: getUser().id,
              token: getUser().token,
              question_id: questionId,
              tagList: JSON.stringify([otherTag]),
            };
            //删除“其他”标签
            removeTagByQuestion(removeData).then((res) => {
              let removeResponse = res.data;
              if (removeResponse.ErrorCode === 1) {
                alert("删除'其他标签'失败");
              } else {
                alert("流转成功");
                location.reload();
              }
            });
          }
        });
      }
    },

    showSelectTags(question) {
      question.tagsListForShow = [];
      question.tags.sort((a, b) => {
        return a.id - b.id;
      });
      let idx = 0;
      this.tagsList.forEach(function (item) {
        if (idx < question.tags.length) {
          if (question.tags[idx].id !== item.id) {
            question.tagsListForShow.push("id:" + item.id + "-" + item.name);
          } else {
            idx++;
          }
        } else {
          question.tagsListForShow.push("id:" + item.id + "-" + item.name);
        }
      });
      return question;
    },

    onTagsInit(tagsTree) {
      //当标签获取完成时，向home组件发出事件，传递标签树，展平
      let that = this;
      let dfs = function (obj) {
        let parentName = obj.name;
        // eslint-disable-next-line no-unused-vars
        obj.children.forEach((item) => {
          item.name = parentName + "/" + item.name;
          dfs(item);
        });
        that.tagsList.push(obj);
      };

      tagsTree.forEach((item) => {
        dfs(item);
      });
      //排序
      that.tagsList.sort((a, b) => {
        return a.id - b.id;
      });

      this.$store.commit("setTagsList", this.tagsList);
      setTagList(this.tagsList);
    },

    //退回问题到两办
    returnBack(questionId, reason) {
      //console.log(1);
      const data = {
        id: getUser().id,
        token: getUser().token,
        question_id: questionId,
      };
      let tagList = [];
      getTagByQuestion(data).then((res) => {
        const response = res.data;
        if (response.ErrorCode === 1) {
          alert("获取问题所属标签失败");
        } else {
          const tagData = response.data;
          for (let i = 0; i < tagData.length; i++) {
            tagList.push(tagData[i].id);
          }
          tagList = JSON.stringify(tagList);

          const removeData = {
            id: getUser().id,
            token: getUser().token,
            question_id: questionId,
            tagList: tagList,
          };

          //删除该所属问题下所有标签
          removeTagByQuestion(removeData).then((res) => {
            const removeResponse = res.data;
            if (removeResponse.ErrorCode === 1) {
              alert("删除问题标签失败");
            } else {
              //这里增加"其他"标签
              //然后还要写退回原因
              //console.log(1);
              const addData = {
                id: getUser().id,
                token: getUser().token,
                question_id: questionId,
                tagList: JSON.stringify([this.searchTagId("其他")]),
                reason: reason,
              };
              addQuestionTag(addData).then((res) => {
                const addResponse = res.data;
                //console.log(addResponse);
                if (addResponse.ErrorCode === 1) {
                  alert("退回失败");
                } else {
                  alert("退回成功");
                  location.reload();
                }
              });
            }
          });
        }
      });
    },

    //通过问题获取答复
    getAnswerByQuestion(questionId, i) {
      const data = {
        question_id: questionId,
        user_id: 0,
      };
      getAnswerByQuestion(data).then((res) => {
        const response = res.data;
        if (response.ErrorCode === 1) {
          alert("拉取数据失败");
          console.log(1);
        } else {
          this.currentQuestions[i].answer = response.data;
        }
      });
    },

    //通过问题获取评论
    getCommentByQuestion(questionId, i) {
      const data = {
        question_id: questionId,
        user_id: 0,
      };
      getCommitByQuestion(data).then((res) => {
        const response = res.data;
        if (response.ErrorCode === 1) {
          alert("拉取数据失败");
          console.log(2);
        } else {
          this.currentQuestions[i].commit = response.data;
        }
      });
    },

    //子传父，获取管理员回复的内容
    getAnswerByChild(comment, questionId) {
      const data = {
        id: getUser().id,
        token: getUser().token,
        question_id: questionId,
        answer_contain: comment,
      };

      addComment(data).then((res) => {
        const response = res.data;
        if (response.ErrorCode === 1) {
          alert("回复失败");
        } else {
          alert("回复成功");
          location.reload();
        }
      });
    },

    getReason: function (questionId, reason) {
      this.returnBack(questionId, reason);
    },

    //通过标签名找标签ID
    searchTagId(tag_name) {
      const tagsList = this.tagsList;
      for (let i = 0; i < tagsList.length; i++) {
        if (tagsList[i].name === tag_name) {
          return tagsList[i].id;
        }
      }
      return -1;
    },
  },
  created() {},
};
</script>

<style scoped>
.card {
  border-radius: 10px;
  border: 1px solid #dddddd;
  box-shadow: 7px 7px 10px rgba(200, 200, 200, 0.2);
  margin: 14px 0;
}
.main-container {
  max-width: 1280px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 20% 80%;
}
.content {
  position: relative;
  margin-top: 50px;
}

.content-title {
  width: 100%;
  color: #000000;
  font-size: 24px;
  text-align: center;
}

.content-text {
  font-size: 16px;
  line-height: 30px;
  text-indent: 32px;
}

p {
  line-height: 180%;
  text-indent: 20px;
}
.question-content {
  position: relative;
}

.select-control >>> .v-menu__content {
  top: 70px !important;
  left: 12px !important;
}

.btn-font-style {
  color: #ffffff;
  font-size: 16px;
  padding: 7px;
  border-radius: 10px;
}
.green {
  background: #66bb6a;
}
.red {
  background: #e53935;
}

.admin-student-icon {
  width: 100%;
  display: flex;
  margin-top: 15px;
  margin-bottom: 15px;
}

#question-block {
  margin-top: 0;
}

.pagination-box {
  margin: 20px;
}

.expansion-box {
  width: 100%;
}

.question-title {
  font-size: 26px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 10px;
  padding: 0 14px 14px 14px;
}

.each-question-box {
  padding: 20px;
}

.each-question-description {
  font-size: 18px;
}

.datetime-style {
  text-align: center;
  color: #999999;
  position: relative;
}

.each-question-content {
  position: relative;
  font-size: 16px;
  line-height: 30px;
  padding: 14px 0;
}
.item-tags {
  padding: 14px 0;
}
@media screen and (max-width: 625px) {
  .content {
    margin: 0;
  }
  .main-container {
    display: flex;
    flex-direction: column;
  }
  .container {
    padding: 0 12px;
  }
  .each-question-box {
    margin: 14px 0;
    padding: 14px;
  }
  .each-question-description {
    padding: 14px 0;
    font-size: 16px;
  }
  .each-question-content {
    font-size: 16px;
    line-height: 28px;
    padding: 0;
  }
  .datetime-style {
    left: 0;
    text-align: center;
  }
}
</style>
