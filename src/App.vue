<template>
  <div class="container column">
    <form class="card card-w30">
      <div class="form-control">
        <label for="type">Тип блока</label>
        <select id="type" v-model="selectedType">
          <option value="title">Заголовок</option>
          <option value="subtitle">Подзаголовок</option>
          <option value="avatar">Аватар</option>
          <option value="text">Текст</option>
        </select>
      </div>

      <div
        class="form-control"
        v-if="selectedType !== 'avatar' && selectedType !== 'subtitle'"
      >
        <label for="value">Значение</label>
        <textarea id="value" rows="3" v-model="inputValue"></textarea>
      </div>

      <div class="form-control" v-else-if="selectedType === 'avatar'">
        <label for="avatarUrl">Ссылка на изображение</label>
        <input id="avatarUrl" type="text" v-model="avatarUrl" />
      </div>

      <div class="form-control" v-else>
        <label for="value">Текст</label>
        <textarea id="value" rows="3" v-model="inputValue"></textarea>
      </div>

      <button class="btn primary" @click.prevent="addItem">Добавить</button>
    </form>
    <div class="card card-w70">
      <h1 v-if="titles?.length === 0">
        Добавьте первый блок, чтобы увидеть результат
      </h1>
      <h1 v-if="titles?.length > 0">{{ titles[0] }}</h1>
      <!-- Displaying only the initial title -->

      <!-- Displaying additional titles -->
      <h1 v-for="(title, index) in additionalTitles" :key="index">
        {{ title }}
      </h1>

      <!-- Displaying the avatar if available -->
      <div v-if="avatarVisible" class="avatar">
        <img :src="avatarUrl" alt="Avatar" />
      </div>

      <!-- Displaying additional subtitles -->
      <div v-for="(item, index) in subtitlesAndText" :key="index">
        <h2 v-if="item.type === 'subtitle'">{{ item.value }}</h2>
        <h3 v-else>{{ item.value }}</h3>
      </div>
    </div>
  </div>

  <div class="container">
    <p>
      <button class="btn primary" @click="loadComments">
        Загрузить комментарии
      </button>
    </p>

    <div v-if="commentsLoading" class="loader"></div>

    <div v-if="!commentsLoading && comments.length > 0" class="card">
      <h2>Комментарии</h2>
      <ul class="list">
        <li class="list-item" v-for="(comment, index) in comments" :key="index">
          <div>
            <p>
              <strong>{{ comment.email }}</strong>
            </p>
            <small>{{ comment.body }}</small>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedType: "title",
      inputValue: "",
      items: [], // Array to store additional text items
      titles: [], // Array to store titles
      subtitles: [], // Array to store subtitles
      avatarUrl: "",
      avatarVisible: false, // флаг для отображения аватара
      comments: [],
      commentsLoading: false,
    };
  },
  computed: {
    subtitlesAndText() {
      // Создаем новый массив, в котором каждый подзаголовок сопоставляется соответствующему текстовому элементу
      const mergedArray = [];
      for (
        let i = 0;
        i < Math.max(this.subtitles.length, this.items.length);
        i++
      ) {
        if (this.subtitles[i]) {
          mergedArray.push({ type: "subtitle", value: this.subtitles[i] });
        }
        if (this.items[i]) {
          mergedArray.push(this.items[i]);
        }
      }
      return mergedArray;
    },
    additionalTitles() {
      return this.titles.slice(1); // Return additional titles excluding the first one
    },
    additionalItems() {
      return this.items.filter((item) => item.type === "text"); // Return additional text items
    },
  },
  methods: {
    addItem() {
      if (this.selectedType === "title" && this.titles.length === 0) {
        this.titles.push(this.inputValue.trim());
        this.inputValue = "";
      } else if (
        this.inputValue.trim() !== "" ||
        this.selectedType === "avatar" ||
        this.selectedType === "subtitle"
      ) {
        if (this.selectedType === "subtitle") {
          this.subtitles.push(this.inputValue.trim());
        } else if (this.selectedType === "avatar") {
          // Don't clear inputValue here, retain the avatarUrl
          this.avatarVisible = true; // Set avatarVisible to true only when adding an avatar
          return; // Exit the method after setting avatarVisible to true
        } else {
          this.items.push({
            type: this.selectedType,
            value: this.inputValue,
          });
        }
        this.inputValue = "";
      }
    },
    async loadComments() {
      // Set loading state to true
      this.commentsLoading = true;

      // Delay for one second
      setTimeout(async () => {
        try {
          // Make API call to fetch comments
          const response = await fetch(
            "https://jsonplaceholder.typicode.com/comments?_limit=42"
          );
          const data = await response.json();

          // Set comments data
          this.comments = data;

          // Set loading state to false
          this.commentsLoading = false;
        } catch (error) {
          console.error("Error fetching comments:", error);
          // Handle error if needed
          this.commentsLoading = false;
        }
      }, 1000); // 1 second delay
    },
  },
};
</script>


<style>
.avatar {
  display: flex;
  justify-content: center;
}

.avatar img {
  width: 150px;
  height: auto;
  border-radius: 50%;
}
</style>