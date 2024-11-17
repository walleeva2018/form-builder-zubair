<template>
    <div class="playground-container">
        <!-- Header -->
        <header class="header">
            <h2>Interactive Playground</h2>
            <div class="header-controls">
                <button @click="toggleLayout" class="layout-toggle" v-if="!isMobileScreen">
                    {{ isVerticalLayout ? '⇄' : '⇅' }}
                </button>
            </div>
        </header>

        <div class="playground-content" :class="{ 'vertical-layout': isVerticalLayout || isMobileScreen }">
            <!-- Input Section -->
            <div class="input-section">
                <h3>Input</h3>
                <div class="input-controls">
                    <select v-model="inputType" class="input-type-select">
                        <option value="create-ui">Create-ui</option>
                        <option value="function">Functions</option>
                        <option value="values">Values</option>
                    </select>
                    <button @click="processInput" class="button ac-button is-primary">
                        Re-Generate
                    </button>
                </div>
                <textarea v-model="inputContent" :placeholder="`Write ${inputType}`" class="input-area"></textarea>
            </div>

            <!-- Preview Section -->
            <div class="preview-section">
                <h3>Generated Form</h3>
                <div class="preview-area" :class="inputType">
                    <FormBuilder v-if="isReady" :key="formBuilderKey" :ui="uiData.ui" :initial-value="uiData.value"
                        :schema="uiData.schema" :logic="uiData.js" />
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'
import FormBuilder from '@/components/FormBuilder.vue'


const formBuilderKey = ref(0);
const functionText = ref<any>('')
const isReady = ref(false)
const uiData = ref<any>({
    ui: {},
    schema: {},
    value: {},
    js: {}
})

const isLoading = ref(false)
const loadData = async () => {
    try {
        isLoading.value = true

        const ui = await import(`../forms/all-element/create-ui.json`)
        const schema = await import(`../forms/all-element/schema.json`)
        const value = await import(`../forms/all-element/values.json`)
        const func = await import(`../forms/all-element/function.js`)
        functionText.value = func


        inputContent.value = JSON.stringify(ui.default, null, 2)
        inputType.value = 'create-ui'


        uiData.value.ui = ui.default
        uiData.value.schema = schema.default
        uiData.value.value = value.default
        uiData.value.js = func.useFunc
    } catch (error) {
        console.log(error)
    }
    isLoading.value = false
    isReady.value = true
}

// State
const inputContent = ref<any>('')
const inputType = ref<string>('create-ui')
const isVerticalLayout = ref<boolean>(false)
const isMobileScreen = ref<boolean>(false)




const processInput = (): void => {
    try {
        if (inputType.value === 'create-ui') {
            uiData.value.ui = JSON.parse(inputContent.value);
        } else if (inputType.value === 'values') {
            uiData.value.value = JSON.parse(inputContent.value);
        } else if (inputType.value === 'function') {
            // Parse functions back into an object
            const funcString = inputContent.value;

            // Create and execute the function properly
            const wrapper = new Function(`return ${funcString}`)();
            uiData.value.js = wrapper;
        }
        formBuilderKey.value++;
    } catch (error) {
        console.error('Error processing input:', error);
    }
};


const toggleLayout = (): void => {
    isVerticalLayout.value = !isVerticalLayout.value
}

// Responsive handling
const checkScreenSize = (): void => {
    isMobileScreen.value = window.innerWidth < 768
    if (isMobileScreen.value) {
        isVerticalLayout.value = true
    }
}

watch(inputType, (newType) => {
    if (newType === 'create-ui') {
        inputContent.value = JSON.stringify(uiData.value.ui, null, 2);
    } else if (newType === 'function') {
        const [, func] = Object.entries(functionText.value)[0];
        if (typeof func === 'function') {
            inputContent.value = `${func.toString()}`;
        } else {
            console.error('The value is not a function');
        }



    } else {
        inputContent.value = JSON.stringify(uiData.value.value, null, 2);
    }
});




onMounted(() => {
    checkScreenSize()
    loadData()
    window.addEventListener('resize', checkScreenSize)
})

onUnmounted(() => {
    window.removeEventListener('resize', checkScreenSize)
})
</script>

<style scoped>
.playground-container {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    background-color: #f5f5f5;
    border-bottom: 1px solid #ddd;
    height: 60px;
    margin-top: 60px;
}

.header-controls {
    display: flex;
    gap: 0.5rem;
}

.playground-content {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    padding: 1rem;
    flex: 1;
    overflow: hidden;
}

.playground-content.vertical-layout {
    grid-template-columns: 1fr;
}

.input-section,
.preview-section {
    display: flex;
    flex-direction: column;
    background: white;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
    overflow: hidden;
}

.input-controls,
.preview-controls {
    display: flex;
    justify-content: space-between;
    margin: 0.5rem 0;
}

.input-type-select,
.process-btn,
.layout-toggle,
.theme-toggle {
    padding: 0.5rem 1rem;
    border-radius: 4px;
    border: 1px solid #ddd;
    background: white;
    cursor: pointer;
}

.input-area {
    flex: 1;
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-family: monospace;
    resize: none;
    overflow: auto;
}

.preview-area {
    flex: 1;
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    overflow: auto;
}

.placeholder-content {
    color: #666;
    font-style: italic;
}

h3 {
    margin: 0;
}

.theme-toggle button {
    padding: 8px 12px;
    border-radius: 4px;
    border: 1px solid #ddd;
    cursor: pointer;
}

.theme-toggle button.active {
    background-color: #333;
    color: white;
}

/* Dark mode styles */
:global(body.dark-mode) .playground-container {
    background-color: #1a1a1a;
    color: #fff;
}

:global(body.dark-mode) .input-section,
:global(body.dark-mode) .preview-section,
:global(body.dark-mode) .input-area,
:global(body.dark-mode) .preview-area {
    border-color: #444;
    background-color: #2a2a2a;
    color: #fff;
}

:global(body.dark-mode) .input-type-select,
:global(body.dark-mode) .process-btn {
    background-color: #333;
    color: #fff;
    border-color: #444;
}


/* Mobile styles */
@media (max-width: 768px) {
    .header {
        height: 50px;
        padding: 0.5rem;
    }

    .playground-content {
        padding: 0.5rem;
    }

    .input-section,
    .preview-section {
        padding: 0.5rem;
    }
}
</style>