<template>
  <div class="home">
    <button @click="selectedPage = 'products'">Products</button>
    <button @click="selectedPage = 'categories'">Categories</button>
    <button @click="selectedPage = 'addProduct'">Add Product</button>
    <button @click="selectedPage = 'addCategory'">Add Category</button>

    <div v-if="selectedPage === 'products'">
      <h2>Products:</h2>
      <select v-model="selectedCategory" @change="fetchProductsByCategory">
        <option value="">All Categories</option>
        <option v-for="category in categories" :key="category.id" :value="category.id">
          {{ category.name }}
        </option>
      </select>
      <ul>
        <li v-for="product in products" :key="product.id">
          <h3>{{ product.name }}</h3>
          <p>Price: {{ product.price }}</p>
          <p>Description: {{ product.description }}</p>
          <p>Category ID: {{ product.category.id }}</p>
        </li>
      </ul>
    </div>

    <div v-else-if="selectedPage === 'categories'">
      <h2>Categories:</h2>
      <ul>
        <template v-for="category in categories" :key="category.id">
          <li>{{ category.name }}</li>
          <ul>
            <li v-for="child in category.children" :key="child.id">
              {{ child.name }}
            </li>
          </ul>
        </template>
      </ul>
    </div>

    <div v-else-if="selectedPage === 'addProduct'">
      <h2>Add Product:</h2>
      <input type="text" v-model="newProduct.name" placeholder="Product Name" />
      <input type="number" v-model="newProduct.price" placeholder="Price" />
      <textarea v-model="newProduct.description" placeholder="Description"></textarea>
      <h4>Categories:</h4>
      <div v-for="category in categories" :key="category.id">
        <input type="checkbox" :id="category.id" :value="category.id" v-model="newProduct.categoryIds" />
        <label :for="category.id">{{ category.name }}</label>
      </div>
      <button @click="addProduct">Add Product</button>
    </div>

    <div v-else-if="selectedPage === 'addCategory'">
      <h2>Add Category:</h2>
      <input type="text" v-model="newCategory.name" placeholder="Category Name" />
      <select v-model="newCategory.parentCategoryId">
        <option value="">Select Parent Category</option>
        <option v-for="category in categories" :key="category.id" :value="category.id">
          {{ category.name }}
        </option>
      </select>
      <button @click="addCategory">Add Category</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: 'HomeView',
  data() {
    return {
      selectedPage: 'products',
      products: [],
      categories: [],
      selectedCategory: '',
      newProduct: {
        name: '',
        price: '',
        description: '',
        categoryIds: [],
      },

      newCategory: {
        name: '',
        parentCategoryId: '',
      },
    };
  },
  methods: {
    async fetchProducts() {
      try {
        const response = await axios.get('http://localhost:8000/api/v1/products/');
        this.products = response.data;
      } catch (error) {
        console.error('Error fetching products:', error);
      }
    },
    async fetchProductsByCategory() {
      try {
        let url = 'http://localhost:8000/api/v1/products/';
        if (this.selectedCategory) {
          url += `?category_id=${this.selectedCategory}`;
        }
        const response = await axios.get(url);
        this.products = response.data;
      } catch (error) {
        console.error('Error fetching products by category:', error);
      }
    },
    async fetchCategories() {
      try {
        const response = await axios.get('http://localhost:8000/api/v1/get-list/');
        this.categories = response.data;
      } catch (error) {
        console.error('Error fetching categories:', error);
      }
    },
    async addProduct() {
      try {
        const payload = {
          name: this.newProduct.name,
          description: this.newProduct.description,
          price: this.newProduct.price,
          category: this.newProduct.categoryIds,
        };
        const response = await axios.post('http://localhost:8000/api/v1/products/', payload);
        this.products.push(response.data);
        this.newProduct = {
          name: '',
          price: '',
          description: '',
          categoryIds: [],
        };
      } catch (error) {
        console.error('Error adding product:', error);
      }
    },

    async addCategory() {
      try {
        const payload = {
          name: this.newCategory.name,
          parent_category: this.newCategory.parentCategoryId || null,
        };
        const response = await axios.post('http://localhost:8000/api/v1/create-category/', payload);
        this.categories.push(response.data);
        this.newCategory = {
          name: '',
          parentCategoryId: '',
        };
      } catch (error) {
        console.error('Error adding category:', error);
      }
    },
  

  
  },
  mounted() {
    this.fetchProducts();
    this.fetchCategories();
  },
};
</script>
