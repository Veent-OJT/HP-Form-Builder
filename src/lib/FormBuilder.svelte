<script lang="ts">
    import { dndzone } from 'svelte-dnd-action';
    import Icon from '@iconify/svelte';
    import type { FormData, FormFieldTypes, FieldType } from './types';
    import FormField from '$lib/FormField.svelte';

  
    let formData: FormData = {
      title: 'Untitled Form',
      description: 'Form Description',
      fields: []
    };
  
    const fieldTypes: { type: FieldType; label: string; icon: string }[] = [
      { type: 'shortText', label: 'Short Text', icon: 'material-symbols:short-text' },
      { type: 'longText', label: 'Long Text', icon: 'material-symbols:text-fields' },
      { type: 'email', label: 'Email', icon: 'material-symbols:email-outline' },
      { type: 'phone', label: 'Phone', icon: 'material-symbols:phone' },
      { type: 'number', label: 'Number', icon: 'material-symbols:numbers' },
      { type: 'date', label: 'Date', icon: 'material-symbols:calendar-month' },
      { type: 'multipleChoice', label: 'Multiple Choice', icon: 'material-symbols:radio-button-checked' },
      { type: 'checkbox', label: 'Checkbox', icon: 'material-symbols:check-box' },
      { type: 'dropdown', label: 'Dropdown', icon: 'material-symbols:arrow-drop-down-circle' },
      { type: 'file', label: 'File Upload', icon: 'material-symbols:upload-file' },
      { type: 'name', label: 'Name', icon: 'material-symbols:person' },
      { type: 'price', label: 'Price', icon: 'material-symbols:payments' },
      { type: 'time', label: 'Time', icon: 'material-symbols:schedule' },
      { type: 'address', label: 'Address', icon: 'material-symbols:location-on' },
    ];
  
    function addField(type: FieldType) {
      const newField: FormFieldTypes = {
        id: crypto.randomUUID(),
        type,
        label: `New ${type} field`,
        required: false,
        options: type === 'multipleChoice' || type === 'checkbox' || type === 'dropdown' ? ['Option 1'] : undefined
      };
      formData.fields = [...formData.fields, newField];
    }
  
    function handleDnd(e: CustomEvent<{ items: FormFieldTypes[] }>) {
      formData.fields = e.detail.items;
    }
  
    function deleteField(id: string) {
      formData.fields = formData.fields.filter(field => field.id !== id);
    }
  
    function updateField(updatedField: FormFieldTypes) {
      formData.fields = formData.fields.map(field => 
        field.id === updatedField.id ? updatedField : field
      );
    }
  </script>
  
  <div class="max-w-4xl mx-auto p-4">
    <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
      <input
        class="text-3xl font-bold w-full mb-2 p-2 border-b-2 border-transparent focus:border-blue-500 focus:outline-none"
        bind:value={formData.title}
        placeholder="Form Title"
      />
      <textarea
        class="w-full p-2 border-b-2 border-transparent focus:border-blue-500 focus:outline-none"
        bind:value={formData.description}
        placeholder="Form Description"
      ></textarea>
    </div>
  
    <div
      use:dndzone={{ items: formData.fields, flipDurationMs: 200 }}
      on:consider={handleDnd}
      on:finalize={handleDnd}
      class="space-y-4"
    >
      {#each formData.fields as field (field.id)}
        <FormField
          {field}
          on:delete={() => deleteField(field.id)}
          on:update={e => updateField(e.detail)}
        />
      {/each}
    </div>
  
    <div class="bg-white rounded-lg shadow-lg p-6 mt-6">
      <h3 class="text-lg font-semibold mb-4">Add Field</h3>
      <div class="grid grid-cols-3 md:grid-cols-4 gap-4">
        {#each fieldTypes as { type, label, icon }}
          <button
            class="flex flex-col items-center p-4 border rounded-lg hover:bg-gray-50 transition-colors"
            on:click={() => addField(type)}
          >
            <Icon {icon} class="text-2xl mb-2" />
            <span class="text-sm">{label}</span>
          </button>
        {/each}
      </div>
    </div>
  </div>
  
  <style>
    :global(.dndzone) {
      min-height: 60px;
    }
  </style>
