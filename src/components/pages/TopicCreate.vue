<!-- view -->
<template>
  <div>
    <App-Header />

    <div id="topic-create">
      <h3>Create a new topic about {{category}}</h3>

      <form v-on:submit="handleSubmit">
        <input type="text" maxLength="80" placeholder="Topic Title" class="title" v-model="title" required/>

        <div class="style-buttons">
          <button type="button" v-bind:class="{active: bold}" v-on:click="styleText('bold')">Bold</button>
          <button type="button" v-bind:class="{active: italic}" v-on:click="styleText('italic')">Italic</button>
          <button type="button" v-bind:class="{active: underline}" v-on:click="styleText('underline')">Underline</button>
          <button type="button" v-bind:class="{active: spoiler}" v-on:click="styleText('spoiler')">Spoiler</button>
        </div>

        <textarea placeholder="Opening Post" ref="post" class="post" v-model="textInput" required></textarea>
        <button type="submit" class="submit">Submit</button>
      </form>
    </div>
  </div>
</template>

<!-- controller -->
<script>
  import Header from "../partials/Header.vue";
  import store from "../../store/index.js";
  import Axios from "axios";
  import jwt from "jsonwebtoken";
  import jwtsecret from "../../../jwtsecret.js"

  export default {
    components: {
      "App-Header": Header
    },
    data: function(){
      return {
        category: this.$route.params.category.toUpperCase()[0] + this.$route.params.category.slice(1),
        title: "",
        textInput: "",
        username: "",
        bold: false,
        italic: false,
        underline: false,
        spoiler: false
      }
    },
    created: function(){
      try{
        var userdata = jwt.verify(store.getState().user.authToken, jwtsecret.secret);
        this.username = userdata.username
      }
      catch(err){
        this.$router.push("/login");
      }
    },
    methods: {
      handleSubmit: function(e){
        e.preventDefault();

        var data = {
          title: this.title,
          message: this.textInput,
          category: this.$route.params.category,
          username: this.username
        }

        Axios.post("/api/topics/create-topic", data)
        .then(result => {
          this.$router.push("/");
        })
        .catch(err => {
          console.log(err);
        })
      },
      styleText: function(type){
        var textArea = this.$refs.post;
        var startPos = textArea.selectionStart;
        var endPos = textArea.selectionEnd;
        var tmpText= textArea.value;

        switch(type){
          case "bold":
            this.textInput = tmpText.substring(0, startPos) + (!this.bold ? "<b>" : "</b>") + tmpText.substring(endPos, tmpText.length);
            this.bold = !this.bold;
            break;
          case "italic":
            this.textInput = tmpText.substring(0, startPos) + (!this.italic ? "<i>" : "</i>") + tmpText.substring(endPos, tmpText.length);
            this.italic = !this.italic;
            break;
          case "underline":
            this.textInput = tmpText.substring(0, startPos) + (!this.underline ? "<u>" : "</u>") + tmpText.substring(endPos, tmpText.length);
            this.underline = !this.underline;
            break;
          case "spoiler":
            this.textInput = tmpText.substring(0, startPos) + (!this.spoiler ? "<span class='spoiler'>" : "</span>") + tmpText.substring(endPos, tmpText.length);
            this.spoiler = !this.spoiler;
            break;
        }
        this.$refs.post.focus();
      },
    }
  }
</script>
