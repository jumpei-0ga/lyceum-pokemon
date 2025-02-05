<script setup>
const route = useRoute();
const router = useRouter();
const config = useRuntimeConfig();
const url = ref('https://pokeapi.co/api/v2/pokemon/');
const { data: pokemons, refresh } = await useFetch(url);
const fetchData = async (newUrl) => {
  if (!newUrl) return;
  url.value = newUrl;
  await refresh();
};
const page = ref(1);
const limit = ref(20);
const maxPage = computed(() => Math.floor(pokemons.value.count / limit.value));
const onNext = async (newUrl) => {
  fetchData(newUrl);
  page.value++;
};
const onPrev = async (newUrl) => {
  fetchData(newUrl);
  page.value--;
};
const onCatch = async (pokemon) => {
  const response = await $fetch(`/api/trainer/${route.params.name}/pokemon`, {
    baseURL: config.public.backendOrigin,
    method: "POST",
    body: {
      name: pokemon.name,
    },
  }).catch((e) => e);
  if (response instanceof Error) return;
  router.push(`/trainer/${route.params.name}`);
};
const { dialog, onOpen, onClose } = useDialog();
</script>

<template>
<div>
  <h1>ポケモンをつかまえる</h1>
    <h3>{{ pokemons.count }}しゅるいのポケモン</h3>
    <h3>{{ page }} / {{ maxPage }} ページ</h3>
    <GamifyList>
      <GamifyItem v-for="pokemon in pokemons.results" :key="pokemon.name">
        <p>{{ pokemon.name }}</p>
        <GamifyButton @click="onOpen(pokemon)">つかまえる</GamifyButton>
      </GamifyItem>
    </GamifyList>
    <GamifyDialog
      v-if="dialog"
      id="confirm-catch"
      title="かくにん"
      :description="`ほう！　${dialog.name}　にするんじゃな？`"
      @close="onClose"
    >
      <GamifyList :border="false" direction="horizon">
        <GamifyItem>
          <GamifyButton @click="onClose">いいえ</GamifyButton>
        </GamifyItem>
        <GamifyItem>
          <GamifyButton @click="onCatch(dialog)">はい</GamifyButton>
        </GamifyItem>
      </GamifyList>
    </GamifyDialog>
    <GamifyList :border="true" direction="horizon">
      <GamifyItem>
        <GamifyButton :disabled="!pokemons.previous" @click="onPrev(pokemons.previous)">まえへ</GamifyButton>
      </GamifyItem>
      <GamifyItem>
        <GamifyButton :disabled="!pokemons.next" @click="onNext(pokemons.next)">つぎへ</GamifyButton>
      </GamifyItem>
    </GamifyList>
</div>
</template>