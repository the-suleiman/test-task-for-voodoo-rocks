<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md">
      <q-input v-model="search" filled type="search" placeholder="Filter by author...">
        <template v-slot:append>
          <q-icon v-if="search === ''" name="search"/>
          <q-icon v-else name="clear" class="cursor-pointer" @click="search = ''"/>
        </template>
      </q-input>
    </div>
    <div>
      <div class="q-pa-md row items-start q-gutter-md">

        <q-card bordered class="my-card" v-for="post in searchedAuthors" :key="post.id">
          <q-card-section>
            <div class="text-h6">{{ post.title }}</div>
            <div class="text-subtitle2">by {{ post.author }}</div>
          </q-card-section>

          <q-separator inset/>

          <q-card-section>
            {{ post.body }}
          </q-card-section>
        </q-card>

      </div>
    </div>
  </q-page>
</template>

<script>

import {api} from "boot/axios";
import {useQuasar} from "quasar";
import {computed, ref} from "vue";

export default {
  setup() {
    const $q = useQuasar()

    let posts = ref([])
    let authors = ref([])
    let res = ref([])
    let search = ref('')

    const postsFetch = async () => {
      try {
        $q.loading.show({message: 'Loading posts...'})
        await api('posts').then(function (response) {
          posts.value = (response.data)
        })
        await api('users').then(function (response) {
          authors.value = (response.data)
        })
        res.value = posts.value.reduce((returnArray, item) => {
          let match = authors.value.find(el => el.id === item.userId);
          if (match) {
            returnArray.push({...item, author: match.name});
          }
          return returnArray;
        }, [])
        $q.loading.hide()
      } catch (e) {
        console.error(e)
      }
    }
    postsFetch()

    const searchedAuthors = computed(() => {
      return res.value.filter((item) => {
        return (
          item.author
            .toLowerCase()
            .indexOf(search.value.toLowerCase()) !== -1
        );
      });
    });

    return {
      search,
      searchedAuthors
    }
  }
}
</script>

<style lang="sass" scoped>
.my-card
  width: 100%
  max-width: 250px
</style>
