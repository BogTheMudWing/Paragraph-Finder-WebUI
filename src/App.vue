<template>
  <div id="app">
    <div class="container">
      <h1>Paragraph Finder</h1>

      <!-- Form -->
      <form @submit.prevent="findParagraph">
        <div class="block">
          <div class="form-group">
            <label for="bookName"><strong>Book Name: </strong></label>
            <select
              id="bookName"
              v-model="bookName"
              @change="fetchChapters"
              class="form-control"
              required
            >
              <option value="" disabled selected>Select a book</option>
              <option v-for="book in books" :key="book" :value="book">
                {{ book }}
              </option>
            </select>
          </div>
        </div>

        <div class="block">
          <div class="form-group">
            <label for="chapter"><strong>Chapter: </strong></label>
            <select
              id="chapter"
              v-model="chapter"
              class="form-control"
              required
              :disabled="!chapters.length"
            >
              <option value="" disabled selected>Select a chapter</option>
              <option v-for="ch in chapters" :key="ch" :value="ch">
                {{ ch }}
              </option>
            </select>
          </div>
        </div>

        <div class="block">
          <div class="form-group">
            <label for="paragraph"><strong>Paragraph: </strong></label>
            <textarea
              id="paragraph"
              v-model="paragraph"
              class="form-control"
              rows="4"
              required
              placeholder="Enter the full paragraph to search for"
            ></textarea>
          </div>
        </div>

        <button type="submit" class="btn btn-primary">Find Match</button>
      </form>

      <!-- Results -->
      <div v-if="result" class="result mt-4">
        <h2>Paragraph Found</h2>
        <p v-if="result.index">
          <strong>This looks like paragraph number:</strong> {{ result.index }}
        </p>
        <p v-if="result.similarity_score">
          <strong>Similarity Score:</strong> {{ result.similarity_score * 100 }}%
        </p>
        <div v-if="result.match">
          <h3>Paragraph:</h3>
          <p>{{ result.match }}</p>
        </div>
      </div>

      <!-- Error -->
      <div v-if="error" class="error mt-4">
        <p class="text-danger">{{ error }}</p>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from "vue";
import axios from "axios";

interface ParagraphResult {
  index?: number;
  similarity_score?: number;
  match?: string;
  reason?: string;
}

export default defineComponent({
  setup() {
    const bookName = ref<string>("");
    const chapter = ref<string>("");
    const paragraph = ref<string>("");
    const result = ref<ParagraphResult | null>(null);
    const error = ref<string | null>(null);

    const books = ref<string[]>([]);
    const chapters = ref<string[]>([]);

    const apiBase = import.meta.env.VITE_APP_API_URL;

    const fetchBooks = async () => {
      try {
        const response = await axios.get(`${apiBase}/books`);
        books.value = response.data.available_books || [];
      } catch (err) {
        console.error("Failed to fetch books", err);
        error.value = "Failed to load books. Please try again later.";
      }
    };

    const fetchChapters = async () => {
      chapters.value = [];
      chapter.value = "";
      if (!bookName.value) return;

      try {
        const response = await axios.get(`${apiBase}/books/${bookName.value}/chapters`);
        chapters.value = response.data.available_chapters || [];
      } catch (err) {
        console.error("Failed to fetch chapters", err);
        error.value = `Could not load chapters for book "${bookName.value}".`;
      }
    };

    const findParagraph = async () => {
      result.value = null;
      error.value = null;

      try {
        const response = await axios.post(apiBase, {
          book_name: bookName.value,
          chapter: chapter.value,
          paragraph: paragraph.value,
        });

        if (response.data.index !== undefined) {
          result.value = response.data;
        } else {
          error.value = "An error occurred: " + response.data.reason;
        }
      } catch (err: any) {
        error.value =
          "An error occurred: " +
          (err.response?.data?.reason || err.message || "Unknown error");
      }
    };

    onMounted(() => {
      fetchBooks();
    });

    return {
      bookName,
      chapter,
      paragraph,
      result,
      error,
      findParagraph,
      books,
      chapters,
      fetchChapters,
    };
  },
});
</script>
