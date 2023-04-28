<script>
export default {
  data: () => ({
    profilePictureUrl: null,
    selectedColor: 'none',
    customFavoriteColor: '',
    childrenCount: 0,
    children: [],
    firstName: '',
    lastName: '',
    personalIdNumber: '',
    personalIdNumberIsValid: null,
    personalIdNumberErrorMessage: '',
  }),

  methods: {
    displayThumbnail (e) {
      this.profilePictureUrl = URL.createObjectURL(e.target.files[0]);
    },
    onSubmit () {
      const output = {
        firstName: this.firstName,
        lastName: this.lastName,
      };
      console.log(JSON.stringify(output));
    },
    validatePersonalIdNumber() {
      if (this.personalIdNumber === '') {
        this.personalIdNumberIsValid = false;
        this.personalIdNumberErrorMessage = 'Rodné číslo je povinné';
      } else if (!isValidCzechPersonalIdNumber(this.personalIdNumber)) {
        this.personalIdNumberIsValid = false;
        this.personalIdNumberErrorMessage = 'Neplatné rodné číslo';
      } else {
        this.personalIdNumberIsValid = true;
      }
    },
    isValidCzechPersonalIdNumber(idNumber) {
      const re = /^(\d{6})(\/)?(\d{3,4})$/;
      const matches = idNumber.match(re);

      if (!matches) {
        return false;
      }

      const birthPart = matches[1];
      const controlPart = matches[3];

      if (controlPart.length === 3) {
        if (parseInt(birthPart, 10) % 11 !== 0) {
          return false;
        }
      } else {
        const fullNumber = parseInt(birthPart + controlPart, 10);

        if (fullNumber % 11 !== 0) {
          if (fullNumber % 11 === 10 && controlPart[3] === '0') {
            return true;
          }
          return false;
        }
      }

      const year = parseInt(birthPart.slice(0, 2), 10) + 1900;
      const month = parseInt(birthPart.slice(2, 4), 10) % 50;
      const day = parseInt(birthPart.slice(4, 6), 10);

      const date = new Date(year, month - 1, day);

      return (
          date.getFullYear() === year &&
          date.getMonth() === month - 1 &&
          date.getDate() === day
      );
    },
    updateChildren() {
      const diff = this.childrenCount - this.children.length;

      if (diff > 0) {
        for (let i = 0; i < diff; i++) {
          this.children.push({ id: Date.now() + i, firstName: '', lastName: '' });
        }
      } else {
        this.children.splice(this.childrenCount);
      }
    },
    addChildren() {
      this.childrenCount++;
    },

  },
  watch: {
    childrenCount() {
      this.updateChildren();
    },

  computed: {
    favoriteColor () {
      return this.selectedColor === 'other'
          ? this.customFavoriteColor
          : this.selectedColor;
    },
    age() {
      if (!this.isValidCzechPersonalIdNumber(this.personalIdNumber)) {
        return null;
      }

      const birthPart = this.personalIdNumber.slice(0, 6);
      const year = parseInt(birthPart.slice(0, 2), 10) + 1900;
      const month = parseInt(birthPart.slice(2, 4), 10) % 50;
      const day = parseInt(birthPart.slice(4, 6), 10);

      const birthDate = new Date(year, month - 1, day);
      const now = new Date();
      const age = now.getFullYear() - birthDate.getFullYear();

      return now.getMonth() < birthDate.getMonth() ||
      (now.getMonth() === birthDate.getMonth() && now.getDate() < birthDate.getDate())
          ? age - 1
          : age;
    },

    gender() {
      if (!this.isValidCzechPersonalIdNumber(this.personalIdNumber)) {
        return null;
      }

      const birthPart = this.personalIdNumber.slice(0, 6);
      const month = parseInt(birthPart.slice(2, 4), 10);

      return month >= 50 ? 'Žena' : 'Muž';
    },
  }
}}
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
  </form>

    <h2>Osobní informace</h2>

    <div class="form-group">
      <label for="firstName">Jméno</label>

      <input
          type="text"
          class="form-control"
          id="firstName"
          v-model="firstName"
      >
    </div>

    <div class="form-group">
      <label for="lastName">Příjmení</label>

      <input
          type="text"
          class="form-control"
          id="lastName"
          v-model="lastName"
      >
    </div>

    <form @submit.prevent="onSubmit">


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

      <small
          class="form-text"
          :style="{ 'color': favoriteColor, 'font-weight': 'bold' }"
      >
        Vaše oblíbená barve je: {{ favoriteColor }}
      </small>
    </div>

    <div class="form-group">
      <label for="personalIdNumber">Rodné číslo</label>

      <input
          type="text"
          :class="['form-control', personalIdNumberIsValid === true ? 'is-valid' : (personalIdNumberIsValid === false ? 'is-invalid' : '')]"
          id="personalIdNumber"
          placeholder="890101/1234"
          v-model="personalIdNumber"
          @blur="validatePersonalIdNumber"
      >

      <div class="invalid-feedback">
        {{ personalIdNumberErrorMessage }}
      </div>

      <small class="form-text text-muted">
        Věk: {{ age }},
        Pohlaví: {{ gender }}
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
