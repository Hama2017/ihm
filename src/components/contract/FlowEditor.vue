// src/components/contract/FlowEditor.vue
<template>
  <div class="flow-editor h-full bg-gray-200 relative">
    <vue-flow
      v-model:elements="elements"
      :default-zoom="1"
      :min-zoom="0.5"
      :max-zoom="2"
    >
      <template #node-state="nodeProps">
        <StateNode :label="nodeProps.data.label" />
      </template>
      
      <template #edge-transition="edgeProps">
        <BaseEdge
          :id="edgeProps.id"
          :path="(edgeProps as any).path"
          :marker-end="(edgeProps as any).markerEnd"
          class="transition-edge"
          :style="{ strokeWidth: '2px', stroke: '#374151' }"
        />
        <EdgeLabelRenderer>
          <div
            class="absolute px-2 py-1 bg-white text-indigo-900 text-sm rounded-md nodrag pointer-events-none"
            :style="{
              transform: `translate(-50%, -50%) translate(${(edgeProps as any).labelX || 0}px, ${(edgeProps as any).labelY || 0}px)`
            }"
          >
            {{ edgeProps.data.label }}
          </div>
        </EdgeLabelRenderer>
      </template>
      
      <Background pattern-color="#aaa" :gap="20" :size="1" />
      
      <Controls />
      <MiniMap />
    </vue-flow>
    
    <!-- Boutons flottants pour ajouter des états et transitions -->
    <div class="absolute top-4 right-4 bg-white p-2 rounded shadow-md space-y-2">
      <button 
        @click="openAddStateModal"
        class="w-full bg-indigo-700 text-white px-3 py-2 rounded hover:bg-indigo-800"
      >
        + État
      </button>
      <button 
        @click="openAddTransitionModal"
        class="w-full bg-gray-700 text-white px-3 py-2 rounded hover:bg-gray-800"
      >
        + Transition
      </button>
    </div>
    
    <!-- Modal pour ajouter un état -->
    <div v-if="showAddStateModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
      <div class="bg-white rounded-lg p-6 w-96">
        <h3 class="text-lg font-semibold mb-4">Ajouter un nouvel état</h3>
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-700 mb-1">Nom de l'état:</label>
          <input 
            v-model="newStateName" 
            type="text" 
            class="w-full px-3 py-2 border border-gray-300 rounded-md"
            placeholder="Ex: État A"
          />
        </div>
        <div class="flex justify-end space-x-2">
          <button @click="closeAddStateModal" class="px-4 py-2 border border-gray-300 rounded-md text-sm">
            Annuler
          </button>
          <button @click="addState" class="px-4 py-2 bg-indigo-700 text-white rounded-md text-sm">
            Ajouter
          </button>
        </div>
      </div>
    </div>
    
    <!-- Modal pour ajouter une transition -->
    <div v-if="showAddTransitionModal" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
      <div class="bg-white rounded-lg p-6 w-96">
        <h3 class="text-lg font-semibold mb-4">Ajouter une transition</h3>
        <div class="mb-3">
          <label class="block text-sm font-medium text-gray-700 mb-1">État source:</label>
          <select 
            v-model="newTransition.source" 
            class="w-full px-3 py-2 border border-gray-300 rounded-md"
          >
            <option value="" disabled>Sélectionner un état</option>
            <option v-for="state in stateNodes" :key="state.id" :value="state.id">
              {{ state.data.label }}
            </option>
          </select>
        </div>
        <div class="mb-3">
          <label class="block text-sm font-medium text-gray-700 mb-1">État destination:</label>
          <select 
            v-model="newTransition.target" 
            class="w-full px-3 py-2 border border-gray-300 rounded-md"
          >
            <option value="" disabled>Sélectionner un état</option>
            <option v-for="state in stateNodes" :key="state.id" :value="state.id">
              {{ state.data.label }}
            </option>
          </select>
        </div>
        <div class="mb-4">
          <label class="block text-sm font-medium text-gray-700 mb-1">Nom de la fonction:</label>
          <input 
            v-model="newTransition.label" 
            type="text" 
            class="w-full px-3 py-2 border border-gray-300 rounded-md"
            placeholder="Ex: aller"
          />
        </div>
        <div class="flex justify-end space-x-2">
          <button @click="closeAddTransitionModal" class="px-4 py-2 border border-gray-300 rounded-md text-sm">
            Annuler
          </button>
          <button @click="addTransition" class="px-4 py-2 bg-indigo-700 text-white rounded-md text-sm">
            Ajouter
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">


import { ref, computed, onMounted, watch } from 'vue';
import { 
  VueFlow, 
  useVueFlow,
  BaseEdge,
  EdgeLabelRenderer,
  type Node,
  type Edge
} from '@vue-flow/core';

import { Background } from '@vue-flow/background';
import { Controls } from '@vue-flow/controls';
import { MiniMap } from '@vue-flow/minimap';

import StateNode from './StateNode.vue';
import type { State, Transition } from '../../types/contract';
// Props
const props = defineProps<{
  states: State[];
  transitions: Transition[];
}>();

const emit = defineEmits<{
  (e: 'update:states', states: State[]): void;
  (e: 'update:transitions', transitions: Transition[]): void;
}>();

// État local
const showAddStateModal = ref(false);
const showAddTransitionModal = ref(false);
const newStateName = ref('');
const newTransition = ref<{
  source: string;
  target: string;
  label: string;
}>({
  source: '',
  target: '',
  label: ''
});

// Initialisation de Vue Flow
const { fitView } = useVueFlow();

// Conversion des états et transitions en éléments pour Vue Flow
const elements = ref<(Node | Edge)[]>([]);

// Calculer les nœuds d'état actuels
const stateNodes = computed(() => {
  return elements.value.filter(el => el.type === 'state') as Node[];
});

// Mise à jour des éléments en fonction des états et transitions
function updateFlowElements() {
  const nodes: Node[] = props.states.map((state, index) => ({
    id: state.id,
    type: 'state',
    position: calculatePosition(index, props.states.length),
    data: { label: state.label }
  }));
  
  const edges: Edge[] = props.transitions.map(transition => ({
    id: transition.id,
    source: transition.source,
    target: transition.target,
    type: 'transition',
    data: { label: transition.label }
  }));
  
  elements.value = [...nodes, ...edges];
  
  // Ajuster la vue
  setTimeout(() => fitView(), 10);
}

// Calculer une position pour un état
function calculatePosition(index: number, total: number): { x: number; y: number } {
  const radius = Math.max(200, 100 * total / 2);
  const angleStep = (2 * Math.PI) / Math.max(total, 1);
  const angle = index * angleStep;
  
  return {
    x: 350 + radius * Math.cos(angle),
    y: 250 + radius * Math.sin(angle)
  };
}

// Modales pour ajouter des états et transitions
function openAddStateModal() {
  showAddStateModal.value = true;
  newStateName.value = '';
}

function closeAddStateModal() {
  showAddStateModal.value = false;
}

function addState() {
  if (!newStateName.value) {
    alert('Veuillez entrer un nom pour l\'état');
    return;
  }
  
  const newState: State = {
    id: `state-${Date.now()}`,
    label: newStateName.value
  };
  
  emit('update:states', [...props.states, newState]);
  closeAddStateModal();
}

function openAddTransitionModal() {
  if (props.states.length < 2) {
    alert('Vous avez besoin d\'au moins deux états pour créer une transition');
    return;
  }
  
  showAddTransitionModal.value = true;
  newTransition.value = {
    source: '',
    target: '',
    label: ''
  };
}

function closeAddTransitionModal() {
  showAddTransitionModal.value = false;
}

function addTransition() {
  if (!newTransition.value.source || !newTransition.value.target || !newTransition.value.label) {
    alert('Veuillez remplir tous les champs');
    return;
  }
  
  if (newTransition.value.source === newTransition.value.target) {
    alert('La source et la destination ne peuvent pas être identiques');
    return;
  }
  
  const newTrans: Transition = {
    id: `transition-${Date.now()}`,
    source: newTransition.value.source,
    target: newTransition.value.target,
    label: newTransition.value.label
  };
  
  emit('update:transitions', [...props.transitions, newTrans]);
  closeAddTransitionModal();
}

// Observer les changements dans les props
watch(() => [props.states, props.transitions], () => {
  updateFlowElements();
}, { deep: true, immediate: true });

// Initialiser la vue au montage
onMounted(() => {
  updateFlowElements();
  setTimeout(() => fitView(), 100);
});
</script>

<style scoped>
.flow-editor {
  width: 100%;
  height: 100%;
  min-height: 600px;
}
</style>