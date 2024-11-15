<template>
  <div class="min-h-screen bg-gradient-to-r from-blue-500 to-teal-400 flex flex-col items-center justify-center text-gray-800 px-4 py-8">
    <!-- Заголовок с градиентом и тенью -->
    <h1 class="text-5xl font-extrabold text-white mb-8 shadow-xl px-10 py-5 rounded-full bg-gradient-to-r from-blue-400 to-teal-500 drop-shadow-2xl">
      Укоротитель ссылок
    </h1>

    <!-- Контейнер для формы -->
    <div class="w-full max-w-lg p-8 bg-white shadow-2xl rounded-3xl transform transition-all hover:scale-105 duration-300 ease-in-out">
      <!-- Форма для ввода URL -->
      <form @submit.prevent="shortenUrl" class="flex flex-col gap-6">
        <input
            v-model="originalUrl"
            type="url"
            class="w-full p-4 border-2 border-blue-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:outline-none text-lg shadow-md transition-all ease-in-out"
        />
        <button
            type="submit"
            class="w-full bg-gradient-to-r from-blue-400 to-teal-500 text-white py-3 rounded-xl font-semibold shadow-xl hover:from-blue-500 hover:to-teal-600 transition-all duration-200"
        >
          Shorten URL
        </button>
      </form>

      <!-- Вывод сокращенной ссылки -->
      <div v-if="shortUrl" class="mt-6 text-center">
        <p class="text-lg font-medium text-blue-600">Введите ссылку которую нужно укоротить:</p>
        <a
            :href="shortUrl"
            target="_blank"
            class="text-blue-500 hover:text-blue-700 underline text-xl"
        >
          {{ shortUrl }}
        </a>
      </div>
    </div>

    <!-- Таблица всех ссылок -->
    <div v-if="links.length" class="w-full max-w-3xl mt-12 bg-white shadow-2xl rounded-3xl p-8">
      <h2 class="text-3xl font-semibold text-blue-600 mb-6">Укороченные ссылки</h2>
      <table class="min-w-full table-auto">
        <thead>
        <tr class="border-b">
          <th class="px-4 py-2 text-left text-lg font-semibold text-blue-600">Укороченная ссылка</th>
          <th class="px-4 py-2 text-left text-lg font-semibold text-blue-600">Ориганальная ссылка</th>
          <th class="px-4 py-2 text-left text-lg font-semibold text-blue-600">Кол-во кликов</th>
          <th class="px-4 py-2 text-left text-lg font-semibold text-blue-600">Дейсивия</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="link in links" :key="link.shortUrl" class="border-b hover:bg-blue-50 transition-all duration-200">
          <td class="px-4 py-2">
            <a :href="link.shortUrl" target="_blank" class="text-blue-500 hover:text-blue-700">
              {{ link.shortUrl }}
            </a>
          </td>
          <td class="px-4 py-2 text-gray-600">{{ link.originalUrl }}</td>
          <td class="px-4 py-2 text-gray-600">{{ link.clicks }}</td>
          <td class="px-4 py-2">
            <button
                @click="copyToClipboard(link.shortUrl)"
                class="bg-blue-500 text-white py-1 px-3 rounded-lg text-sm hover:bg-blue-600 focus:outline-none shadow-md transform transition-all hover:scale-105 duration-200"
            >
              <font-awesome-icon :icon="['fas', 'copy']" class="text-white" />
            </button>
          </td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { library } from "@fortawesome/fontawesome-svg-core";
import { faCopy } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

// Добавление иконки в библиотеку
library.add(faCopy);

export default {
  components: {
    FontAwesomeIcon,
  },
  data() {
    return {
      originalUrl: "",
      shortUrl: null,
      links: [],
    };
  },
  methods: {
    async shortenUrl() {
      try {
        const response = await axios.post("http://localhost:5000/shorten", {
          originalUrl: this.originalUrl,
        });
        const newLink = {
          shortUrl: response.data.shortUrl,
          originalUrl: this.originalUrl,
        };
        this.links.push(newLink); // Добавляем новую ссылку в список
        this.shortUrl = newLink.shortUrl; // Отображаем последнюю сокращенную ссылку
        this.originalUrl = ""; // Очищаем поле ввода
      } catch (error) {
        console.error(error);
      }
    },

    async fetchLinks() {
      try {
        const response = await axios.get("http://localhost:5000/links");
        this.links = response.data.links; // Получаем все ссылки с сервера
      } catch (error) {
        console.error(error);
      }
    },

    copyToClipboard(url) {
      navigator.clipboard.writeText(url).then(() => {
      }).catch((error) => {
        console.error("Failed to copy URL:", error);
      });
    }
  },

  mounted() {
    this.fetchLinks(); // Загружаем все ссылки при загрузке компонента
  },
};
</script>

<style>
@import "tailwindcss/tailwind.css";

body {
  font-family: 'Inter', sans-serif;
}

h1, h2 {
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.1);
}

button:hover {
  transform: scale(1.05);
}

table td, table th {
  transition: background-color 0.3s;
}

table tbody tr:hover {
  background-color: rgba(59, 130, 246, 0.1);
}

input:focus {
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.5);
}

button {
  transition: background-color 0.2s, transform 0.2s;
}

button:hover {
  background-color: #2b6cb0;
  transform: scale(1.1);
}

table {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

table th, table td {
  padding: 12px;
  text-align: left;
}

table th {
  background-color: #f1f5f9;
}

table td {
  background-color: #ffffff;
}

table tr:nth-child(even) td {
  background-color: #f9fafb;
}

table tr:hover {
  background-color: rgba(59, 130, 246, 0.1);
}

table td a {
  color: #3182ce;
}

table td a:hover {
  color: #2b6cb0;
}
</style>
