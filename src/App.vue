<template>
  <div class="container">
    <b-row>
      <b-col class="col-5">
        <h1>Product Management</h1>
      </b-col>
      <b-col class="col-1 mt-2">
        <div v-if="listLoader" class="loading ml-2">
          <div class="spinner-border" role="status"></div>
        </div>
      </b-col>
      <b-col class="col-3 mt-2 pl-0">
        <div class="input-group">
          <input
              type="search"
              class="form-control rounded"
              placeholder="Search"
              aria-label="Search"
              aria-describedby="search-addon"
          />
          <button type="button" class="btn search-btn-style">Search</button>
        </div>
      </b-col>
      <b-col class="col-3 mt-2 pr-0">
        <b-button variant="success" @click="handelOpenCreatModal"
        >Create New
        </b-button
        >
      </b-col>
    </b-row>
    <table class="table table-bordered table-width">
      <thead>
      <tr>
        <th width="15%">No</th>
        <th width="15%">Name</th>
        <th width="50%">Details</th>
        <th width="20%">Action</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(product, index) in ProductList" v-bind:key="index">
        <td>{{ product.id }}</td>
        <td>{{ product.name }}</td>
        <td>{{ product.details }}</td>
        <td>
          <b-button
              class="margin-button"
              variant="primary"
              @click.prevent.stop="updateProduct(product)"
          >Edit
          </b-button
          >
          <b-button
              class="margin-button"
              variant="danger"
              @click.prevent.stop="deleteProduct(product)"
          >Delete
          </b-button
          >
        </td>
      </tr>
      </tbody>
    </table>
    <b-row>
      <b-col class="col-7"></b-col>
      <b-col class="col-5">
        <div class="pagination-position mt-3">
          <b-pagination
              v-model="currentPage"
              :total-rows="rows"
              :per-page="perPage"
              first-number
              last-number
              @change="pagination"
          ></b-pagination>
        </div>
      </b-col>
    </b-row>
    <!-- creatModal -->
    <b-modal
        id="creat-modal"
        ref="creat-modal"
        class="modal-backdrop"
        title="Creat New"
        :hide-footer="true"
    >
      <form
          @submit.prevent="handleSubmitProduct"
          class="form-modal-custom-style"
      >
        <b-form-group label="Name" label-for="name" class="input-modal-champ">
          <b-form-input
              id="Name"
              v-model="productToAdd.name"
              required
          ></b-form-input>
        </b-form-group>
        <b-form-group label="Details" label-for="details">
          <b-form-textarea
              id="details"
              v-model="productToAdd.details"
              required
          ></b-form-textarea>
        </b-form-group>
        <div class="d-flex justify-content-center mt-2">
          <b-button type="submit">
            <span>
              Submit
              <div v-if="loader" class="loading ml-2">
                <div class="spinner-border" role="status"></div>
              </div>
            </span>
          </b-button>
        </div>
      </form>
    </b-modal>
    <!-- UpdateModal -->
    <b-modal
        id="creat-modal"
        ref="Edit-modal"
        class="modal-backdrop"
        title="Edit Product"
        :hide-footer="true"
    >
      <form
          @submit.prevent="handleUpdateProduct"
          class="form-modal-custom-style"
      >
        <b-form-group label="Name" label-for="name" class="input-modal-champ">
          <b-form-input id="Name" v-model="productToUpdate.name" required></b-form-input>
        </b-form-group>
        <b-form-group label="Details" label-for="details">
          <b-form-textarea id="details" v-model="productToUpdate.details" required></b-form-textarea>
        </b-form-group>
        <div class="d-flex justify-content-center mt-2">
          <b-button type="submit">
            <span>
              Submit
              <div v-if="loader" class="loading ml-2">
                <div class="spinner-border" role="status"></div>
              </div>
            </span>
          </b-button>
        </div>
      </form>
    </b-modal>
    <!-- deleteModal -->
    <b-modal
        id="delete-modal"
        ref="delete-modal"
        class="modal-backdrop"
        title="Delete Product"
        :hide-footer="true"
    >
      <form
          @submit.prevent="handleDeleteProduct"
          class="form-modal-custom-style"
      >
        <p>Do you really want to delete this Product?</p>
        <div class="d-flex justify-content-center mt-2">
          <b-button type="submit">
            <span>
              Submit
              <div v-if="loader" class="loading ml-2">
                <div class="spinner-border" role="status"></div>
              </div>
            </span>
          </b-button>
        </div>
      </form>
    </b-modal>
  </div>
</template>

<script>
// import { mapActions, mapGetters } from "vuex";
import axios from 'axios'

export default {
  name: "App",
  components: {},
  data() {
    return {
      loader: false,
      listLoader: false,
      rows: null,
      perPage: 3,
      currentPage: 1,
      productToAdd: {
        name: null,
        details: null,
      },
      productToUpdate: {
        name: null,
        details: null,
      },
      productToDelete: null,
      ProductList: [],
    };
  },
  methods: {
    handelOpenCreatModal() {
      this.$refs["creat-modal"].show();
    },
    async handleSubmitProduct() {
      this.loader = true;
      const response = await axios.post(`http://localhost:8080/products`, {
        name: this.productToAdd.name,
        details: this.productToAdd.details
      });
      if (response.status === 201) {
        this.loader = false;
        this.$refs["creat-modal"].hide();
        this.getAllProducts();
      } else {
        this.loader = false;
      }
    },
    updateProduct(productValues) {
      this.productToUpdate = {name: productValues.name, details: productValues.details, id: productValues.id};
      this.$refs["Edit-modal"].show();

    },
    deleteProduct(product) {
      this.$refs["delete-modal"].show();
      this.productToDelete = product;
    },
    async getAllProducts() {
      this.listLoader = true;

      const response = await axios.get(`http://localhost:8080/products?page=${this.currentPage}&per-page=${this.perPage}`);
      this.listLoader = false;
      this.ProductList = response.data.items;
      this.rows = response.data._meta.totalCount;

      return true;


    },
    pagination(paginate) {
      this.currentPage = paginate;

      this.getAllProducts();
    },
    async handleUpdateProduct() {
      const response = await axios.put(`http://localhost:8080/products/${this.productToUpdate.id}`, {
        name: this.productToUpdate.name,
        details: this.productToUpdate.details
      });
      if (response.status === 200) {
        this.$refs["Edit-modal"].hide();
        this.getAllProducts();
      }


    },
    async handleDeleteProduct() {
      const response = await axios.delete(`http://localhost:8080/products/${this.productToDelete.id}`);
      if (response.status === 204) {
        this.getAllProducts();
        this.loader = false;
      } else {
        this.loader = false;
      }

      this.$refs["delete-modal"].hide();
    },
  },
  async mounted() {
    this.getAllProducts();
  }
};
</script>

<style lang="css" scoped>
.search-btn-style {
  background-color: #e6e6e6;
  margin-left: -15px;
}

.search-btn-style:hover {
  background-color: #e6e6e6;
  border: none;
  margin-left: -15px;
}

.pagination-position {
  float: right !important;
  display: flex;
  align-items: center;
  justify-content: center;
}

.margin-button {
  margin-right: 8px;
}
</style>
<style>
.modal-backdrop {
  opacity: 0.5 !important;
}
</style>
