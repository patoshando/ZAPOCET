<template>
  <div class="create-post">
    <BlogCoverPreview v-show="this.$store.state.blogphotoPreview" />
    <BlogCoverPreview v-show="this.$store.state.blogPhotoPreview" />
    <Loading v-show="loading" />
    <div class="container">
      <div :class="{ invisible: !error }" class="err-message">
        <p><span>Error:</span>{{ this.errorMsg }}</p>
      </div>
      <div class="blog-info">
        <input type="text" :placeholder="$t('create-post.titulek-prispevku')" v-model="blogTitle" />
        <div class="upload-file">
          <label for="blog-photo">{{ $t('create-post.cover-obrazek') }}</label>
          <input type="file" ref="blogPhoto" id="blog-photo" @change="fileChange" accept=".png, .jpg, ,jpeg" />
          <button @click="openPreview" class="preview" :class="{ 'button-inactive': !this.$store.state.blogPhotoFileURL }">
            {{ $t('create-post.preview-obrazku') }}
          </button>
          <span>{{ $t('create-post.vybrany-soubor') }} {{ this.$store.state.blogPhotoName }}</span>
        </div>
      </div>
      <div class="editor">
        <vue-editor :editorOptions="editorSettings" v-model="blogHTML" useCustomImageHandler @image-added="imageHandler" />
      </div>
      <div class="blog-actions">
        <button @click="uploadBlog" >{{ $t('create-post.zverejnit-prispevek') }}</button>
        <router-link class="router-button" :to="{name: 'PreviewPrispevek'}">{{ $t('create-post.preview-prispevek') }}</router-link>
      </div>
    </div>
  </div>
</template>

<script>
import Quill from "quill";
import firebase from "firebase/app";
import "firebase/storage";
import Loading from "../components/Loading.vue";
import db from "../firebase/firebaseInit";
import BlogCoverPreview from "../components/BlogCoverPreview.vue";
window.Quill = Quill;
const ImageResize = require("quill-image-resize-module").default;
Quill.register("modules/imageResize", ImageResize);
import i18n from '../i18n';

export default {
    name: "CreatePost",
    data() {
        return {
            file: null,
            error: null,
            errorMsg: null,
            loading: null,
            editorSettings: {
                modules: {
                    imageResize: {},
                },
            },
        };
    },
    methods: {
        fileChange() {
            this.file = this.$refs.blogPhoto.files[0];
            const fileName = this.file.name;
            this.$store.commit("fileNameChange", fileName);
            this.$store.commit("createFileURL", URL.createObjectURL(this.file));
        },
        openPreview(){
          this.$store.commit("openPhotoPreview");
        },
        imageHandler(file, Editor, cursorLocation, resetUploader) {
          const storageRef = firebase.storage().ref();
          const docRef = storageRef.child(`documents/blogPostPhotos/${file.name}`);
          docRef.put(file).on(
            "state_changed",
            (snapshot) => {
              console.log(snapshot);
            },
            (err) => {
              console.log(err);
            },
            async () => {
              const downloadURL = await docRef.getDownloadURL();
              Editor.insertEmbed(cursorLocation, "image", downloadURL);
              resetUploader();
            }
          );
        },
        uploadBlog(){
          if (this.blogTitle.length !== 0 && this.blogHTML.length !== 0 ){
            if (this.file){
              this.loading = true;
              const storageRef = firebase.storage().ref();
              const docRef = storageRef.child(`documents/BlogCoverPhotos/${this.$store.state.blogPhotoName}`);
              docRef.put(this.file).on("state_changed", (snapshot) => {
                console.log(snapshot);
              }, (err) => {
                console.log(err);
                this.loading = false;
              }, async () =>{
                const downloadURL = await docRef.getDownloadURL();
                const timestamp = await Date.now();
                const dataBase = await db.collection("Vydani-casopisu").doc();

                await dataBase.set({
                  blogID: dataBase.id,
                  blogHTML: this.blogHTML,
                  blogCoverPhoto: downloadURL,
                  blogCoverPhotoName: this.blogCoverPhotoName,
                  blogTitle: this.blogTitle,
                  profileId: this.profileId,
                  date: timestamp,
                });
                await this.$store.dispatch("getPost");
                this.loading = false;
                this.$router.push({name: 'ZhlednoutPrispevek', params:{blogid:dataBase.id }})
              }
            );
              return;
            }
            this.error = true;
            this.errorMsg = i18n.t("create-post.error1");
            setTimeout(() => {
              this.error = false;
            }, 5000);
            return;
          }
          this.error = true;
          this.errorMsg = i18n.t("create-post.error2");
          setTimeout(() => {
            this.error = false;
          }, 5000);
          return;
        },
    },
    computed: {
        profileId() {
            return this.$store.state.profileId;
        },
        blogCoverPhotoName() {
            return this.$store.state.blogPhotoName;
        },
        blogTitle: {
          get() {
            return this.$store.state.blogTitle;
          },
          set(payload) {
            this.$store.commit("updateBlogTitle", payload);
          },
        },
        blogHTML: {
            get() {
                return this.$store.state.blogHTML;
            },
            set(payload) {
                this.$store.commit("newBlogPost", payload);
            },
        },
    },
    components: { 
      BlogCoverPreview,
      Loading,
      },
};
</script>

<style lang="scss">
.create-post{
  position: relative;
  margin-left: 250px;
  height: 100%;
  padding-right: 20px;
  @media(max-width: 550px){
      margin-left: initial;
      padding: 10px;
      p,span{
        font-size: initial;
      }
    }

  button{
    margin-top: 0;
  }

  .router-button{
    text-decoration: none;
    color:#fff;
  }


  label,
  button,
  .router-button{
    transition: 0.5s ease-in-out all;
    align-self: center;
    font-size: 14px;
    cursor: pointer;
    border-radius: 20px;
    padding: 12px 24px;
    color: #fff;
    background-color: #303030;
    text-decoration: none;
    
    
    &:hover{
      background-color: rgba(48, 48, 48, 0.7);
    }
  }

  .container{
    position: relative;
    display: initial !important;
    height: 90%;
    padding: 10px 25px 60px;
  }



  // error styling

  .invisible{
    opacity: 0 !important;
  }

  .err-message{
    width: 100%;
    padding: 12px;
    border-radius: 8px;
    color: #fff;
    background-color: #303030;
    opacity: 1;
    transition: 0.5 ease all;
    @media(max-width: 550px){
      margin-bottom: 100px !important;
    }


    p{
      font-size: 14px;
    }

    span{
      font-weight: 600;
    }
  }

  .blog-info{
    display: flex;
    margin-bottom: 32px;

    input:nth-child(1){
      min-width: 300px;
    }

    input{
      transition: .5s ease-in-out all;
      margin-left: 10px;
      padding: 10px 4px;
      border: none;
      border-bottom: #303030;

      &:focus{
        outline: none;
        box-shadow: 0 1px 0 0 #303030;
      }
    }

    .upload-file{
      flex: 1;
      margin-left: 16px;
      position: relative;
      display: flex;
      @media(max-width: 550px){
        margin-left: -290px;
        margin-bottom: 30px;
      }
      

      input{
        display: none;
      }

      .preview{
        margin-left: 16px;
        text-transform: initial;
      }

      span{
        font-size: 12px;
        margin-left: 16px;
        align-self: center;
      }
    }
  }


  .editor{
    height: 60vh;
    margin-right: 300px;
    width: 100%;


    .quillWrapper{
      position: relative;
      display: flex;
      flex-direction: column;
      height: 100%;
    }

    .ql-container{
      display: flex;
      flex-direction: column;
      height: 100%;
      overflow: scroll;
    }

    .ql-editor{
      padding: 20px 16px 30px;
    }
  }


  .blog-actions{
    margin-top: 32px;

    button{
      margin-right: 16px;
    }
  }
}
</style>