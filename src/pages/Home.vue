<template>
    <div class="container mx-auto p-4 flex flex-col items-center justify-center min-h-screen relative">
      <h1 class="text-3xl font-bold mb-6">
        <img src="../assets/img/images.png" alt="Rick and Morty" class="mx-auto" />
      </h1>
  
      <!-- Barra de pesquisa -->
      <div class="flex flex-col items-center justify-center gap-4 mb-6 w-full max-w-lg">
        <div class="relative w-full">
          <input
            v-model="searchQuery.name"
            placeholder="Digite o nome do personagem..."
            class="border-2 border-gray-300 p-3 pl-10 pr-4 rounded-full w-full focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all"
          />
          <!-- Ícone de busca -->
          <span class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-500">
            <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 18l6-6m0 0l-6-6m6 6H4" />
            </svg>
          </span>
        </div>
  
        <!-- Filtro de status -->
        <select v-model="searchQuery.status" class="border-2 border-gray-300 p-3 rounded-full w-full focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all">
          <option value="">Selecione o status</option>
          <option value="Alive">Vivo</option>
          <option value="Dead">Morto</option>
          <option value="unknown">Desconhecido</option>
        </select>
  
        <button
          @click="fetchCharacters(1)"
          class="bg-blue-500 text-white p-3 rounded-full w-full md:w-1/3 hover:bg-blue-600 transition-all duration-300"
        >
          Pesquisar
        </button>
      </div>
  
      <p class="text-lg font-semibold mb-4">
        Total de personagens cadastrados: {{ totalCharacters }}
      </p>
  
      <!-- Lista de Personagens -->
      <div v-if="filteredCharacters.length > 0" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4 mt-4">
        <div v-for="character in filteredCharacters" :key="character.id" class="border p-4 rounded-lg">
          <img :src="character.image" :alt="character.name" class="w-full rounded-lg" />
          <h2 class="text-lg font-bold mt-2">{{ character.name }}</h2>
          <button @click="openModal(character)" class="inline-block text-blue-500 bg-white border-2 border-blue-500 rounded-lg py-1 px-2 hover:bg-blue-500 hover:text-white transition-all">
            Ver detalhes
          </button>
        </div>
      </div>
  
      <!-- Mensagem de "Nenhum personagem encontrado" -->
      <div v-else class="text-center text-red-500 font-bold mt-6">
        Nenhum personagem encontrado para os critérios de pesquisa.
      </div>
  
      <!-- Navegação de Paginação -->
      <div v-if="totalPages > 1" class="mt-6 flex justify-center gap-4">
        <button
          :disabled="currentPage === 1"
          @click="changePage(currentPage - 1)"
          class="px-4 py-2 bg-blue-500 text-white rounded-lg disabled:bg-gray-300 transition-all"
        >
          Anterior
        </button>
        <span class="flex items-center justify-center text-lg font-semibold">
          Página {{ currentPage }} de {{ totalPages }}
        </span>
        <button
          :disabled="currentPage === totalPages"
          @click="changePage(currentPage + 1)"
          class="px-4 py-2 bg-blue-500 text-white rounded-lg disabled:bg-gray-300 transition-all"
        >
          Próxima
        </button>
      </div>
  
      <!-- Modal -->
      <div v-if="showModal" class="fixed inset-0 flex items-center justify-center z-50">
        <!-- Fundo Desfocado -->
        <div class="absolute inset-0 bg-gray-800 opacity-50 backdrop-blur-lg"></div>
  
        <!-- Conteúdo do Modal -->
        <div class="bg-white p-6 rounded-lg w-96 z-10 shadow-lg">
          <h2 class="text-2xl font-bold mb-4">{{ selectedCharacter.name }}</h2>
          <img :src="selectedCharacter.image" :alt="selectedCharacter.name" class="w-full rounded-lg mb-4" />
          <p><strong>Status:</strong> {{ selectedCharacter.status }}</p>
          <p><strong>Total de Episódios:</strong> {{ selectedCharacter.episode.length }}</p>
          <p><strong>Localização</strong>: {{ selectedCharacter.location.name }}</p>
          <button @click="closeModal" class="mt-4 bg-blue-500 text-white py-2 px-4 rounded-lg hover:bg-blue-600">
            Fechar
          </button>
        </div>
      </div>

      <footer class=" text-black text-center py-2 mt-auto pt-7">
        <p class="text-sm">&copy; 2025 Criado por Victor Ferreira</p>
      </footer>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, onMounted, computed } from "vue";
  import api from "../services/api";
  
  // Definindo a interface para os personagens
  interface Character {
    id: number;
    name: string;
    status: string;
    image: string;
    episode: string[];
    location: { name: string };
  }
  
  // Variáveis de estado
  const characters = ref<Character[]>([]);
  const showModal = ref<boolean>(false);
  const selectedCharacter = ref<Character | null>(null);
  const totalCharacters = ref<number>(0);
  const totalPages = ref<number>(0);
  const currentPage = ref<number>(1);
  
  // Variáveis de pesquisa
  const searchQuery = ref({
    name: "",
    status: ""
  });
  
  // Número de personagens por página
  const charactersPerPage = 20;
  
  // Método para buscar os personagens
  const fetchCharacters = async (page: number) => {
    try {
      const response = await api.get("/character", {
        params: {
          page: page,
          name: searchQuery.value.name,
          status: searchQuery.value.status
        }
      });
  
      characters.value = response.data.results;
      totalCharacters.value = response.data.info.count;
      totalPages.value = Math.ceil(totalCharacters.value / charactersPerPage);
      currentPage.value = page;
    } catch (error) {
      characters.value = [];
      totalCharacters.value = 0;
      totalPages.value = 0;
    }
  };
  
  // Computed property para filtrar os personagens
  const filteredCharacters = computed(() => {
    return characters.value.filter(character => {
      const matchName = character.name.toLowerCase().includes(searchQuery.value.name.toLowerCase());
      const matchStatus = searchQuery.value.status ? character.status.toLowerCase() === searchQuery.value.status.toLowerCase() : true;
      return matchName && matchStatus;
    });
  });
  
  // Função para mudar de página
  const changePage = (page: number) => {
    if (page > 0 && page <= totalPages.value) {
      fetchCharacters(page);
    }
  };
  
  // Funções de controle do modal
  const openModal = (character: Character) => {
    selectedCharacter.value = character;
    showModal.value = true;
  };
  
  const closeModal = () => {
    showModal.value = false;
    selectedCharacter.value = null;
  };
  
  // Buscar personagens ao carregar o componente
  onMounted(() => {
    fetchCharacters(1);
  });
  </script>
  
  <style scoped>
  /* Estilo do modal com fundo desfocado */
  .fixed {
    z-index: 9999; /* Garante que o modal fique acima de outros elementos */
  }
  
  /* Aplica um fundo cinza escuro com opacidade e desfoque no fundo */
  .backdrop-blur-lg {
    backdrop-filter: blur(10px); /* Aplica o efeito de desfoque no fundo */
    background-color: rgba(0, 0, 0, 0.5); /* Aplica a transparência com o fundo escuro */
  }
  
  /* Adiciona uma sombra no modal para destacar mais */
  .shadow-lg {
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
  </style>
  