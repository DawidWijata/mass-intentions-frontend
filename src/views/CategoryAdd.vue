<template>
  <form class="category-add" @submit.prevent="saveChanges">
    <div class="title-section">
      <aside class="title">
        <h1>
          Dodawanie nowej kategorii
        </h1>
        <go-back-button />
      </aside>
      <div class="buttons">
        <image-button
          caption="Zapisz"
          image-src="save-icon.svg"
          :task="saveChanges"
        />
      </div>

    </div>
    <div class="inputs">
      <input-with-label
        label="Nazwa"
        name="title"
        input-type="text"
        v-model="category.name"
      />
    </div>
  </form>
</template>

<script>
import InputWithLabel from "@/components/InputWithLabel"
import ImageButton from "@/components/ImageButton"
import GoBackButton from "@/components/GoBackButton";
import {ApiService} from "@/services/api-service"
const APIService = new ApiService()

export default {
  name: "CategoryAdd",
  components: {
    InputWithLabel,
    ImageButton,
    GoBackButton
  },

  data(){
    return {
      category: {
        name: ''
      },
      categories: undefined,
      apiError: 'Błąd 403: nie możesz dodawać kategorii - poproś administratora'
    }
  },

  computed: {
    apiModel: function () {
      return {
        name: this.category.name,
      }
    }
  },

  methods: {
    saveChanges: async function () {
      if(this.categories.find(x => x.name === this.category.name)){
        console.log("Category already exists");
        return;
      }

      const [error] = await APIService.addCategory(this.apiModel);

      if(error){
        console.error(error)
        return
      }

      await this.$router.push('/list')
    },
  },

  async created(){
    const email = encodeURI(localStorage.getItem('email'))
    const [categoryError, categories] = await APIService.getCategories()
    const [roleError, role] = await APIService.getRoleByEmail(email)

    if(roleError || role !== 'Admin'){
      await this.$router.push('/list')
    }

    if(categoryError){
      console.error(this.apiError)
      return
    }

    this.categories = categories
  },
}
</script>

<style scoped>
.title-section{
  display: flex;
  justify-content: space-between;
  height: 60px;
}

.title-section h1{
  font-size: 2rem;
  color: rgba(57, 29, 6, 1);
}

.category-add{
  flex: 1;
  padding: 80px 50px 40px 50px;
  height: 100%;

  display: flex;
  flex-direction: column;
}

.buttons, .title{
  display: flex;
}

.title button{
  align-self: flex-start;
  margin-top: 8px;
}

.inputs{
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  flex: 1;
}
</style>