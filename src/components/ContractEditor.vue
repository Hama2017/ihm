// src/components/ContractEditor.vue
<template>
  <div class="contract-editor">
    <!-- Titre du contrat en haut -->
    <div class="bg-white py-4 text-center">
      <h1 class="text-2xl font-bold text-indigo-900">{{ contract.name }}</h1>
    </div>
    
    <!-- Structure à trois panneaux -->
    <div class="flex h-[calc(100vh-140px)]">
      <!-- Panneau gauche: Liste des automates -->
      <div class="w-64 bg-indigo-700 text-white p-4">
        <h2 class="text-xl font-bold mb-4 text-center">AUTOMATES</h2>
        
        <div class="space-y-2">
          <button 
            v-for="automate in contract.automates" 
            :key="automate.id"
            @click="selectAutomate(automate.id)"
            class="w-full py-2 px-4 rounded-full font-semibold"
            :class="automate.active ? 'bg-indigo-900 text-white' : 'bg-white text-indigo-900'"
          >
            {{ automate.name }}
          </button>
          <button 
            @click="showNewAutomateModal = true" 
            class="w-full py-2 px-4 rounded-full bg-indigo-500 text-white font-semibold mt-6 hover:bg-indigo-600 transition-colors"
          >
            + NOUVEL AUTOMATE
          </button>
        </div>
      </div>
      
      <!-- Panneau central: Éditeur d'automate -->
      <div class="flex-1 bg-gray-200 flex flex-col">
        <div class="p-4 bg-gray-300 font-semibold text-center">
          {{ activeAutomate ? activeAutomate.name : 'Aucun automate sélectionné' }}
        </div>
        <div class="flex-1">
          <FlowEditor 
            v-if="activeAutomate"
            :states="activeAutomate.states"
            :transitions="activeAutomate.transitions"
            @update:states="updateStates"
            @update:transitions="updateTransitions"
          />
          <div v-else class="h-full flex items-center justify-center text-gray-500">
            Sélectionnez un automate ou créez-en un nouveau
          </div>
        </div>
        <div class="p-4 flex justify-end">
          <button 
            @click="deployContract"
            class="bg-indigo-700 text-white px-8 py-2 rounded-md font-semibold hover:bg-indigo-800 transition-colors"
            :disabled="!activeAutomate || activeAutomate.states.length < 2"
          >
            DEPLOYER
          </button>
        </div>
      </div>
      
      <!-- Panneau droit: Liste des fonctions -->
      <div class="w-64 bg-indigo-700 text-white p-4">
        <h2 class="text-xl font-bold mb-4 text-center">FONCTIONS</h2>
        
        <div class="space-y-2">
          <button 
            v-for="transition in activeAutomate?.transitions || []" 
            :key="transition.id"
            class="w-full py-2 px-4 rounded-full bg-white text-indigo-900 font-semibold"
          >
            {{ transition.label }}
          </button>
          <div v-if="!activeAutomate || activeAutomate.transitions.length === 0" class="text-center py-4 text-indigo-200">
            Aucune fonction définie
          </div>
        </div>
      </div>
    </div>
    
    <!-- Modal pour ajouter un nouvel automate -->
    <div v-if="showNewAutomateModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
      <div class="bg-white rounded-lg p-6 w-96">
        <h3 class="text-lg font-semibold mb-4">Ajouter un nouvel automate</h3>
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-700 mb-1">Nom de l'automate:</label>
          <input 
            v-model="newAutomateName" 
            type="text" 
            class="w-full px-3 py-2 border border-gray-300 rounded-md"
            placeholder="Ex: AUTOMATE 06"
          />
        </div>
        <div class="flex justify-end space-x-2">
          <button @click="showNewAutomateModal = false" class="px-4 py-2 border border-gray-300 rounded-md text-sm">
            Annuler
          </button>
          <button @click="addNewAutomate" class="px-4 py-2 bg-indigo-700 text-white rounded-md text-sm">
            Ajouter
          </button>
        </div>
      </div>
    </div>
    
    <!-- Notification de déploiement réussi -->
    <div v-if="deploymentSuccess" class="fixed bottom-4 right-4 bg-green-500 text-white p-4 rounded-md shadow-lg z-50">
      Déploiement réussi! 
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import FlowEditor from './contract/FlowEditor.vue';
import type { Contract, Automate, State, Transition } from '../types/contract';

// État initial du contrat
const contract = ref<Contract>({
  id: '0001',
  name: 'CONTRACT 0001',
  automates: [
    {
      id: '01',
      name: 'ELEMENT 01',
      active: false,
      states: [],
      transitions: []
    },
    {
      id: '02',
      name: 'AUTOMATE 02',
      active: false,
      states: [],
      transitions: []
    },
    {
      id: '03',
      name: 'AUTOMATE 03',
      active: false,
      states: [],
      transitions: []
    },
    {
      id: '04',
      name: 'AUTOMATE 04',
      active: false,
      states: [],
      transitions: []
    },
    {
      id: '05',
      name: 'AUTOMATE 05',
      active: true,
      states: [
        { id: 'etat-a', label: 'État A' },
        { id: 'etat-b', label: 'État B' }
      ],
      transitions: [
        { 
          id: 'transition-1', 
          source: 'etat-a', 
          target: 'etat-b', 
          label: 'aller' 
        }
      ]
    }
  ]
});

// État local
const showNewAutomateModal = ref(false);
const newAutomateName = ref('');
const deploymentSuccess = ref(false);

// Automate actif
const activeAutomate = computed<Automate | undefined>(() => {
  return contract.value.automates.find((a: Automate) => a.active);
});

// Sélectionner un automate
function selectAutomate(id: string) {
  contract.value.automates.forEach((a: Automate) => {
    a.active = a.id === id;
  });
}

// Ajouter un nouvel automate
function addNewAutomate() {
  if (!newAutomateName.value) {
    alert('Veuillez entrer un nom pour l\'automate');
    return;
  }
  
  // Désactiver tous les automates
  contract.value.automates.forEach((a: Automate) => a.active = false);
  
  // Créer et ajouter le nouvel automate
  const newId = (contract.value.automates.length + 1).toString().padStart(2, '0');
  
  const newAutomate: Automate = {
    id: newId,
    name: newAutomateName.value,
    active: true,
    states: [],
    transitions: []
  };
  
  contract.value.automates.push(newAutomate);
  
  // Fermer la modale et réinitialiser
  showNewAutomateModal.value = false;
  newAutomateName.value = '';
}

// Mettre à jour les états
function updateStates(states: State[]) {
  if (activeAutomate.value) {
    activeAutomate.value.states = states;
  }
}

// Mettre à jour les transitions
function updateTransitions(transitions: Transition[]) {
  if (activeAutomate.value) {
    activeAutomate.value.transitions = transitions;
  }
}

// Déployer le contrat
function deployContract() {
  if (!activeAutomate.value) {
    return;
  }
  
  if (activeAutomate.value.states.length < 2) {
    alert('Vous devez définir au moins deux états');
    return;
  }
  
  if (activeAutomate.value.transitions.length === 0) {
    alert('Vous devez définir au moins une transition');
    return;
  }
  
  // Simuler un déploiement réussi
  deploymentSuccess.value = true;
  
  // Cacher la notification après 3 secondes
  setTimeout(() => {
    deploymentSuccess.value = false;
  }, 3000);
  
  // Ici, vous pourriez sauvegarder le contrat dans un fichier JSON
  console.log('Contrat à sauvegarder:', JSON.stringify(contract.value, null, 2));
}
</script>