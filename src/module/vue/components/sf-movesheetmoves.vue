<template>
  <div class="flexcol">
    <div class="flexrow nogrow" style="margin-top: 0.5rem">
      <input
        type="text"
        :placeholder="$t('IRONSWORN.Search')"
        v-model="searchQuery"
      />
      <i
        class="fa fa-times-circle nogrow clickable text"
        @click="clearSearch"
        style="padding: 6px"
      />
    </div>

    <div class="flexcol item-list">
      <div class="nogrow" v-if="searchQuery">
        <sf-moverow
          v-for="move of searchResults"
          :key="move.dfid"
          :actor="actor"
          :move="move"
          @moveclick="highlightMove"
        />
      </div>
      <div class="nogrow" v-else v-for="category of categories" :key="category.$id">
        <h2>
          {{ category.tname }}
        </h2>
        <sf-moverow
          v-for="move of category.Moves"
          :key="move.dfid"
          :actor="actor"
          :move="move"
          @moveclick="highlightMove"
        />
      </div>
    </div>
  </div>
</template>

<style lang="less" scoped>
h2 {
  margin: 0.5rem 0 0.3rem;
}
.item-list {
  padding: 0 0.5rem;
}
</style>

<script>
export default {
  props: {
    actor: Object,
  },

  data() {
    return {
      searchQuery: '',
      categories: [],
    }
  },

  async created() {
    const pack = game.packs.get('foundry-ironsworn.starforgedmoves')
    const compendiumMoves = await pack.getDocuments()

    const categories = CONFIG.IRONSWORN.Dataforged.moves
    for (const category of categories) {
      // Provide an i18n str for category names
      category.tname = this.$t(`IRONSWORN.${category.Name}`)

      for (const move of category.Moves) {
        // Provide a Foundry move
        const foundryItem = compendiumMoves.find(
          (x) => x.data.data.dfid === move.$id
        )
        move.foundryItem = foundryItem?.toObject(true)
        move.highlighted = false
      }
    }
    this.categories = categories
  },

  computed: {
    sortedMoves() {
      const ret = []
      for (const category of this.categories) {
        ret.push(...category.Moves)
      }
      return ret
    },

    searchResults() {
      if (!this.searchQuery) return null

      const re = new RegExp(this.searchQuery, 'i')
      return this.sortedMoves.filter((x) => re.test(x.foundryItem.name))
    },
  },

  methods: {
    movesForKey(ck) {
      return this.moves[ck]?.moves
    },

    clearSearch() {
      this.searchQuery = ''
    },

    async highlightMove(item) {
      this.searchQuery = ''
      await new Promise((r) => setTimeout(r, 10))
      // TODO: this doesn't support custom moves
      for (const category of this.categories) {
        for (const move of category.Moves) {
          if (move.$id === item.data.data.dfid) {
            move.highlighted = true
            setTimeout(() => (move.highlighted = false), 2000)
            return
          }
        }
      }

      // Not found; just open the sheet
      item.sheet?.render(true)
    },
  },
}
</script>
