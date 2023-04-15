<script>
export default {
  data: () => ({
    profilePictureUrl: null,
    selectedColor: 'none',
    customFavoriteColor: '',
    childrenCount: 0,
    children: [],
  }),

  methods: {
    displayThumbnail (e) {
      this.profilePictureUrl = URL.createObjectURL(e.target.files[0]);
    }
  },

  computed: {
    favoriteColor () {
      return this.selectedColor === 'other'
          ? this.customFavoriteColor
          : this.selectedColor;
    }
  }
}
</script>

<template>
  <h1 class="mb-4">Úvod do VueJS - 1. tematická práce</h1>

  <form>
    <figure class="figure">
      <img
          v-if="profilePictureUrl !== null"
          :src="profilePictureUrl"
          class="figure-img img-fluid rounded"
          alt="Profile picture"
          @click="profilePictureUrl = null"
      >

      <!--
          Po zvolení obrázku se placeholder hodnota nahradí za preview skutečného obrázku.
          Hint: URL.createObjectURL()
      -->
      <div class="custom-file">
        <input
            type="file"
            class="custom-file-input"
            id="profilePicture"
            @change="displayThumbnail"
        >

        <label
            class="custom-file-label"
            for="profilePicture"
        >
          Změnit profilový obrázek
        </label>
      </div>
    </figure>

    <h2>Osobní informace</h2>

    <div class="form-group">
      <label for="firstName">Jméno</label>

      <input
          type="text"
          class="form-control"
          id="firstName"
      >
    </div>

    <div class="form-group">
      <label for="lastName">Příjmení</label>

      <input
          type="text"
          class="form-control"
          id="lastName"
      >
    </div>

    <div class="form-group">
      <label for="favoriteColor">Oblíbená barva</label>

      <select
          v-model="selectedColor"
          class="form-control"
          id="favoriteColor"
      >
        <option value="none">-- Zvolte --</option>
        <option value="blue">Modrá</option>
        <option value="red">Červená</option>
        <option value="white">Bílá</option>
        <option value="black">Černá</option>
        <option value="green">Zelená</option>
        <option value="other">Jiná</option>
      </select>

      <!-- Vastní oblíbenou barvu zadá uživatel pouze tehdy zvolí-li v selectu výše možnost "Jiná" -->
      <div v-show="selectedColor === 'other'">
        <label class="mt-2" for="customFavoriteColor">Zadejte vlastní oblíbenou barvu:</label>

        <input
            v-model="customFavoriteColor"
            type="text"
            class="form-control"
            placeholder="#ff00ff"
            id="customFavoriteColor"
        >
      </div>

      <!--
          Kromě toho, že se do tohoto textu vypíše zvolená barva,
          slovo barvy (př.: Zelená) bude mít barvu písma odpovídající zvolené barvě.
      -->
      <small
          class="form-text"
          :style="{ 'color': favoriteColor, 'font-weight': 'bold' }"
      >
        Vaše oblíbená barve je: {{ favoriteColor }}
      </small>
    </div>

    <div class="form-group">
      <label for="personalIdNumber">Rodné číslo</label>

      <!--
        Po validaci obdrží input třídu is-valid resp. is-invalid.
        Validace nastane při @blur události.
      -->
      <input
          type="text"
          class="form-control is-invalid"
          id="personalIdNumber"
          placeholder="890101/1234"
      >

      <div class="invalid-feedback">
        <!--
          Validační hláška bude dynamická v závislosti na chybě, která nastala.
          Možné chyby:
              - rodné číslo nezadáno
              - neplatné rodné číslo dle https://cs.wikipedia.org/wiki/Rodn%C3%A9_%C4%8D%C3%ADslo#Kontroln%C3%AD_%C4%8D%C3%ADslice
        -->
        Rodné číslo je povinné
      </div>

      <small class="form-text text-muted">
        <!-- Tyto hodnoty budou dynamicky vypočteny z rodného čísla -->
        Věk: 18,
        Pohlaví: Muž/Žena
      </small>
    </div>

    <div class="form-group">
      <label for="childrenCount">Počet dětí</label>

      <!-- Výchozí hodnota bude 0 -->
      <input
          v-model.number="childrenCount"
          type="number"
          class="form-control"
          id="childrenCount"
          min="0"
      >

      {{ childrenCount }}
    </div>

    <!-- Pokud bude počet dětí > 0 sobrazí se textové inputy pro každé dítě -->

    <div v-show="childrenCount > 0">
      <h2>Děti</h2>

      <div
          v-for="(child, index) in children"
          class="my-2"
          :key="child.id"
      >
        Dítě {{ index + 1 }}:

        <div class="form-row">
          <div class="col">
            <input
                v-model="child.firstName"
                type="text"
                class="form-control"
                placeholder="Jméno"
            >
          </div>

          <div class="col">
            <input
                v-model="child.lastName"
                type="text"
                class="form-control"
                placeholder="Příjmení"
            >
          </div>
        </div>
      </div>

      <button
          type="button"
          class="btn btn-primary"
          @click="addChildren"
      >
        Add child
      </button>

      <pre><code>{{ children }}</code></pre>
    </div>

    <h2>Trvalý pobyt</h2>

    <div class="form-row">
      <div class="col-md-6 mb-3">
        <label for="city">Město</label>
        <input
            type="text"
            class="form-control"
            id="city"
            required
        >
      </div>

      <div class="col-md-6 mb-6">
        <label for="street">Ulice</label>
        <input
            type="text"
            class="form-control"
            id="street"
            required
        >
      </div>
    </div>

    <!-- Pokud uživatel zaškrtne tuto možnost, zobrazí se mu další dvě pole pro korespondenční adresu -->
    <div class="form-group form-check">
      <input
          type="checkbox"
          class="form-check-input"
          id="differentMailingAddress"
      >

      <label class="form-check-label" for="differentMailingAddress">
        Korespondenční adresa se liší od trvalé
      </label>
    </div>

    <h2>Korespondenční adresa</h2>

    <div class="form-row">
      <div class="col-md-6 mb-3">
        <label for="mailingAddressCity">Město</label>
        <input
            type="text"
            class="form-control"
            id="mailingAddressCity"
            required
        >
      </div>

      <div class="col-md-6 mb-6">
        <label for="mailingAddressStreet">Ulice</label>
        <input
            type="text"
            class="form-control"
            id="mailingAddressStreet"
            required
        >
      </div>
    </div>

    <h2>Zákonný zástupce</h2>

    <!-- Pole pro zákonného zástupce se zobrazí pouze tehdy, je-li věk nižší než 18 -->
    <div class="form-group">
      <label for="legalRepresentativeFirstName">Jméno</label>

      <input
          type="text"
          class="form-control"
          id="legalRepresentativeFirstName"
      >
    </div>

    <div class="form-group">
      <label for="legalRepresentativeLastName">Příjmení</label>

      <input
          type="text"
          class="form-control"
          id="legalRepresentativeLastName"
      >
    </div>

    <h2>Ostatní</h2>

    <div class="form-group">
      <label for="note">Poznámka</label>
      <textarea class="form-control" id="note" rows="3"></textarea>
      <!-- Automaticky dopočítávaná hodnota, jakmile se dostane na 5 a méně znaků zčervená -->
      <small class="text-right form-text text-muted">Využito 42/100 znaků</small>
    </div>

    <button
        type="submit"
        class="btn btn-primary"
    >
      Odeslat
    </button>
  </form>

  <h2>Odeslaná data:</h2>

  <pre><code>
{
    "firstName": "John",
    "lastName": "Doe",
    "favoriteColor": "green",
    "personalIdNumber": "890101/1234",
    "children": [
        { "firstName": "Jinny", "lastName": "Doe" },
        { "firstName": "James": "lastName": "Doe" }
    ]
    "city": "Ústí nad Labem",
    "street": "Masarykova",
    "mailingAddress": {
        "city": "Prague",
        "street": "Pařížská"
    },
    "legalRepresentative": {
        "firstName": "Bob",
        "lastName": "Doe"
    },
    "note": "Lorem ipsum dolor sit amet"
}
          </code></pre>

</template>

<style scoped>
</style>
