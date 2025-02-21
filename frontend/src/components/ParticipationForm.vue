<template>
  <div class="form-container">
    <h2>Enregistrer une participation</h2>

    <!-- Sélection de la personne -->
    <div class="form-group">
      <label>Personne</label>
      <select v-model="selectedPersonne">
        <option :value="null" disabled>-- Choisir une personne --</option>
        <option
          v-for="pers in personnes"
          :key="pers.matricule"
          :value="pers.matricule"
        >
          {{ pers.nom }} {{ pers.prenom }}
        </option>
      </select>
    </div>

    <!-- Sélection du projet -->
    <div class="form-group">
      <label>Projet</label>
      <select v-model="selectedProjet">
        <option :value="null" disabled>-- Choisir un projet --</option>
        <option
          v-for="proj in projets"
          :key="proj.id"
          :value="proj.id"
        >
          {{ proj.nom }}
        </option>
      </select>
    </div>

    <!-- Rôle -->
    <div class="form-group">
      <label>Rôle</label>
      <input type="text" v-model="role" />
    </div>

    <!-- Pourcentage (slider) -->
    <div class="form-group">
      <label>Pourcentage</label>
      <div class="slider-container">
        <input
          type="range"
          min="0"
          max="100"
          step="1"
          v-model="pourcentage"
        />
        <span class="slider-value">{{ pourcentage }}%</span>
      </div>
    </div>

    <!-- Bouton Valider -->
    <button class="btn-submit" @click="enregistrerParticipation">Enregistrer</button>

    <!-- Affichage de l'erreur si besoin -->
    <div v-if="errorMessage" class="error-message">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, toRefs } from 'vue'
import doAjaxRequest from '@/util/util.js'


const state = reactive({
  personnes: [],
  projets: [],
  selectedPersonne: null,
  selectedProjet: null,
  role: '',
  pourcentage: 0,
  errorMessage: '',
})


const {
  personnes,
  projets,
  selectedPersonne,
  selectedProjet,
  role,
  pourcentage,
  errorMessage
} = toRefs(state)


function loadPersonnes() {
  doAjaxRequest('http://localhost:8989/api/personnes')
    .then((res) => {
      state.personnes = res._embedded ? res._embedded.personnes : []
    })
    .catch((err) => {
      state.errorMessage = err.message
    })
}


function loadProjets() {
  doAjaxRequest('http://localhost:8989/api/projets')
    .then((res) => {
      state.projets = res._embedded ? res._embedded.projets : []
    })
    .catch((err) => {
      state.errorMessage = err.message
    })
}


function enregistrerParticipation() {
  state.errorMessage = ''

  if (!state.selectedPersonne || !state.selectedProjet) {
    state.errorMessage = 'Sélectionnez une personne et un projet.'
    return
  }
  if (!state.role) {
    state.errorMessage = 'Précisez le rôle.'
    return
  }
  if (state.pourcentage <= 0) {
    state.errorMessage = 'Le pourcentage doit être > 0.'
    return
  }

  const fraction = state.pourcentage / 100

  const url =
    `http://localhost:8989/api/gestion/participation?matricule=${state.selectedPersonne}` +
    `&codeProjet=${state.selectedProjet}` +
    `&role=${encodeURIComponent(state.role)}` +
    `&pourcentage=${fraction}`

  doAjaxRequest(url, { method: 'POST' })
    .then((result) => {
      console.log('Réponse du serveur :', result)
      alert('Participation enregistrée avec succès !')
      state.selectedPersonne = null
      state.selectedProjet = null
      state.role = ''
      state.pourcentage = 0
    })
    .catch((err) => {
      state.errorMessage = err.message
    })
}

onMounted(() => {
  loadPersonnes()
  loadProjets()
})
</script>

<style scoped>
.form-container {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* Titre */
.form-container h2 {
  margin-bottom: 1.5rem;
  font-size: 1.2rem;
  font-weight: bold;
}

/* Groupes de champs */
.form-group {
  margin-bottom: 1rem;
}

/* Labels */
.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 600;
}

/* Select et input text */
.form-group select,
.form-group input[type="text"] {
  width: 100%;
  padding: 0.5rem;
  font-size: 0.9rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Container pour le slider et la valeur */
.slider-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

/* Slider (range) */
.slider-container input[type="range"] {
  flex: 1;
}

/* Valeur du pourcentage */
.slider-value {
  font-weight: 500;
}

/* Bouton Enregistrer */
.btn-submit {
  display: inline-block;
  padding: 0.6rem 1.2rem;
  font-size: 0.9rem;
  color: #fff;
  background-color: #007BFF; /* Bleu style bootstrap */
  border-radius: 4px;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.btn-submit:hover {
  background-color: #0056b3;
}

/* Erreur en rouge */
.error-message {
  margin-top: 1rem;
  color: red;
}
</style>
