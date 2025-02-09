<template>
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="/src/style.css" rel="stylesheet">
  </head>
  <body>
    <div class="container mx-auto p-4 max-w-7xl">
    <h1 class="text-3xl font-bold mb-8 text-center text-indigo-600">Gerenciador de Produtos</h1>

    <!-- Campo de Busca e Botão -->
    <div class="mb-6 flex space-x-4 justify-center">
      <input v-model="searchQuery" type="text" placeholder="Buscar produto..." class="border p-3 rounded-lg w-1/2 shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" />
      <button @click="searchProduct" class="bg-indigo-600 text-white p-3 rounded-lg shadow-md hover:bg-indigo-700 transition-colors">Buscar</button>
    </div>

    <div v-if="products.length === 0" class="mb-4 text-center text-gray-500">
      <p>Sem produtos cadastrados</p>
    </div>

    <div v-for="product in products" :key="product.id" class="mb-6 p-6 border rounded-lg shadow-lg hover:shadow-xl transition-shadow">
      <h2 class="text-2xl font-semibold text-indigo-600">{{ product.nome }}</h2>
      <p class="product-description text-white my-2 p-3 rounded-lg">{{ product.descricao }}</p>
      <p class="product-price text-white font-semibold p-3 rounded-lg"> Preço: ${{ product.preco }}</p>
      <div class="mt-4 flex space-x-4">
        <button @click="editProduct(product)" class="bg-blue-600 text-white p-3 rounded-lg shadow-md hover:bg-blue-700 transition-colors">Editar</button>
        <button class="delete-button bg-red-600 text-white p-3 rounded-lg shadow-md hover:bg-red-700 transition-colors" @click="deleteProduct(product.id)">Deletar</button>
      </div>
    </div>

    <!-- Formulário de Adição de Produto -->
    <div class="mt-8">
      <h2 class="text-2xl text-white font-semibold mb-4 text-gray-800">Inserir novo produto</h2>
      <form @submit.prevent="createProduct" class="flex flex-col space-y-4">
        <input v-model="newProduct.nome" type="text" placeholder="Nome do produto" class="border p-3 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required />
        <input v-model="newProduct.preco" type="number" placeholder="Preço" class="border p-3 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required />
        <textarea v-model="newProduct.descricao" placeholder="Descrição" class="border p-3 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required></textarea>
        <button type="submit" class="add-button bg-green-600 text-white p-3 rounded-lg shadow-md hover:bg-green-700 transition-colors"> Adicionar Produto </button>
      </form>
    </div>

    <!-- Formulário de Edição -->
    <div v-if="productToUpdate.id !== null" class="mt-8">
      <h2 class="text-2xl text-white font-semibold mb-4 text-gray-800">Editar Produto</h2>
      <form @submit.prevent="updateProduct" class="flex flex-col space-y-4">
        <input v-model="productToUpdate.nome" type="text" placeholder="Nome do produto" class="border p-3 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required />
        <input v-model="productToUpdate.preco" type="number" placeholder="Preço" class="border p-3 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required />
        <textarea v-model="productToUpdate.descricao" placeholder="Descrição" class="border p-3 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required></textarea>
        <button type="submit" :disabled="isLoading" class="update-button bg-green-600 text-white p-3 rounded-lg shadow-md hover:bg-green-700 transition-colors">
          <span v-if="isLoading">Carregando...</span>
          <span v-else>Atualizar Produto</span>
        </button>
      </form>
    </div>
  </div>
  </body>
  </html>
</template>

<style scoped>
@tailwind base;
@tailwind components;
@tailwind utilities;
</style>

<script>
import { ref, onMounted } from "vue";

export default {
  setup() {
    const products = ref([]);
    const newProduct = ref({
      nome: "",
      preco: 0,
      descricao: ""
    });
    const searchQuery = ref(""); // Variável para armazenar o valor da busca
    const productToUpdate = ref({
      id: null,
      nome: "",
      preco: 0,
      descricao: ""
    });
    const isLoading = ref(false); // Estado de carregamento

    // Fetch all products from the API
    const fetchProducts = async () => {
      try {
        const response = await fetch("http://localhost:3000/products");
        products.value = await response.json();
      } catch (error) {
        console.error("Error fetching products:", error);
        alert("Erro ao carregar os produtos. Tente novamente.");
      }
    };

    // Create a new product
    const createProduct = async () => {
      try {
        const response = await fetch("http://localhost:3000/products", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(newProduct.value)
        });
        const createdProduct = await response.json();
        products.value.push(createdProduct);
        newProduct.value = { nome: "", preco: 0, descricao: "" }; // Reset form
      } catch (error) {
        console.error("Error creating product:", error);
        alert("Erro ao criar o produto. Tente novamente.");
      }
    };

    // Delete a product
    const deleteProduct = async (id) => {
      try {
        await fetch(`http://localhost:3000/products/${id}`, {
          method: "DELETE"
        });
        products.value = products.value.filter((product) => product.id !== id);
      } catch (error) {
        console.error("Error deleting product:", error);
        alert("Erro ao deletar o produto. Tente novamente.");
      }
    };

    // Set the product to update
    const editProduct = (product) => {
      productToUpdate.value = { ...product }; // Preenche os dados do produto para edição
    };

    // Validação do formulário de atualização
    const validateForm = () => {
      if (!productToUpdate.value.nome || !productToUpdate.value.preco || !productToUpdate.value.descricao) {
        alert("Todos os campos são obrigatórios!");
        return false;
      }
      if (productToUpdate.value.preco <= 0) {
        alert("O preço deve ser um valor positivo.");
        return false;
      }
      return true;
    };

    // Update a product
    const updateProduct = async () => {
      if (!validateForm()) {
        return;
      }

      isLoading.value = true;

      try {
        const response = await fetch(`http://localhost:3000/products/${productToUpdate.value.id}`, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            nome: productToUpdate.value.nome,
            preco: productToUpdate.value.preco,
            descricao: productToUpdate.value.descricao
          })
        });

        if (!response.ok) {
          throw new Error("Erro ao atualizar o produto");
        }

        const updatedProduct = await response.json();
        console.log("Produto atualizado com sucesso:", updatedProduct);

        // Feedback para o usuário
        alert("Produto atualizado com sucesso!");

        // Opcional: Resetar o formulário ou esconder a seção de edição
        productToUpdate.value = { id: null, nome: "", preco: "", descricao: "" };

        // Aqui você pode recarregar a lista de produtos, se necessário
        // await fetchProducts();
      } catch (error) {
        console.error("Erro ao atualizar o produto:", error);
        alert("Erro ao atualizar o produto. Tente novamente.");
      } finally {
        isLoading.value = false;
      }
    };

    // Realiza a busca de produtos
    const searchProduct = async () => {
      if (searchQuery.value.trim()) {
        try {
          const response = await fetch(`http://localhost:3000/products?nome_like=${searchQuery.value}`);
          products.value = await response.json();
        } catch (error) {
          console.error("Erro ao buscar produtos:", error);
          alert("Erro ao buscar os produtos. Tente novamente.");
        }
      } else {
        fetchProducts(); // Recarrega todos os produtos quando a busca estiver vazia
      }
    };

    onMounted(fetchProducts);

    return {
      products,
      newProduct,
      searchQuery,
      productToUpdate,
      isLoading,
      fetchProducts,
      createProduct,
      deleteProduct,
      editProduct,
      updateProduct,
      searchProduct
    };
  }
};
</script>
