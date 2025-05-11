<template>
  <div id="app">
    <div class="container">
      <h1>Paragraph Finder</h1>

      <!-- Form -->
      <form @submit.prevent="findParagraph">
        <div class="block">
          <div class="form-group">
            <label for="bookName"><strong>Book Name: </strong></label>
            <input
              type="text"
              id="bookName"
              v-model="bookName"
              class="form-control"
              required
              placeholder="Enter the book name"
            />
          </div>
          <label class="form-description">This should be same as the article name on the wiki.</label>
        </div>

        <div class="block">
          <div class="form-group">
            <label for="chapter"><strong>Chapter: </strong></label>
            <input
              type="text"
              id="chapter"
              v-model="chapter"
              class="form-control"
              required
              placeholder="Enter chapter"
            />
          </div>
          <label class="form-description">This should be a number or a title like prologue.</label>
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
import { defineComponent, ref } from "vue";
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

    const findParagraph = async () => {
      result.value = null;
      error.value = null;

      try {
        const response = await axios.post(import.meta.env.VITE_APP_API_URL, {
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

    return {
      bookName,
      chapter,
      paragraph,
      result,
      error,
      findParagraph,
    };
  },
});
</script>