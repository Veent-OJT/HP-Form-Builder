<script lang="ts">
    import { dndzone } from 'svelte-dnd-action';
    import Icon from '@iconify/svelte';
    import type { FormData, FormFieldTypes, FieldType } from './types';
    import FormField from '$lib/FormField.svelte';

    let isPreviewMode = false;
    let formResponses: Record<string, any> = {};
  
    let formData: FormData = {
      title: 'Registration Form',
      description: 'Please fill out the form below',
      fields: [
        {
          id: crypto.randomUUID(),
          type: 'name',
          label: 'Name',
          required: true,
        },
        {
          id: crypto.randomUUID(),
          type: 'email',
          label: 'Email Address',
          required: true,
        },
        {
          id: crypto.randomUUID(),
          type: 'phone',
          label: 'Contact Number',
          required: true,
        }
      ]
    };
  
    const fieldTypes: { type: FieldType; label: string; icon: string }[] = [
      { type: 'shortText', label: 'Short Text', icon: 'material-symbols:short-text' },
      { type: 'longText', label: 'Long Text', icon: 'material-symbols:text-fields' },
      { type: 'email', label: 'Email', icon: 'material-symbols:mail-outline' },
      { type: 'phone', label: 'Phone', icon: 'material-symbols:call' },
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

    function togglePreview() {
      isPreviewMode = !isPreviewMode;
      if (isPreviewMode) {
        formResponses = {};
      }
    }

    function handleSubmit() {
      console.log('Form responses:', formResponses);
      alert("submitted")
    }
</script>
  
<div class="max-w-4xl mx-auto p-4">
  <div class="bg-white rounded-lg shadow-lg p-6">
    {#if !isPreviewMode}
      <!-- Builder Mode -->
      <h1>Registration prompt text</h1>
      <input
        class="text-3xl font-bold w-full mb-2 p-2 border-b-2 border-transparent focus:border-blue-500 focus:outline-none"
        placeholder="Form Title"
        bind:value={formData.title}
      />
      <textarea
        class="w-full p-2 border-b-2 border-transparent focus:border-blue-500 focus:outline-none mb-6"
        bind:value={formData.description}
        placeholder="Form Description"
      ></textarea>

      <div
        use:dndzone={{ items: formData.fields, flipDurationMs: 200 }}
        on:consider={handleDnd}
        on:finalize={handleDnd}
        class="space-y-4 mb-6"
      >
        {#each formData.fields as field (field.id)}
          <FormField
            {field}
            on:delete={() => deleteField(field.id)}
            on:update={e => updateField(e.detail)}
          />
        {/each}
      </div>

      <div class="mt-6">
        <h3 class="text-lg font-semibold mb-4">Add Field</h3>
        <div class="grid grid-cols-3 md:grid-cols-4 gap-4">
          {#each fieldTypes as { type, label, icon }}
            <button
              class="cursor-pointer flex flex-col items-center p-4 border rounded-lg hover:bg-gray-50 transition-colors"
              on:click={() => addField(type)}
            >
              <Icon {icon} class="text-2xl mb-2" />
              <span class="text-sm">{label}</span>
            </button>
          {/each}
        </div>
      </div>

    {:else}
      <!-- Preview Mode -->
      <form on:submit|preventDefault={handleSubmit} class="space-y-6">
        <h1 class="text-3xl font-bold mb-2">{formData.title}</h1>
        <p class="text-gray-600 mb-6">{formData.description}</p>

        {#each formData.fields as field (field.id)}
          <div class="space-y-2">
            <label class="block text-sm font-medium text-gray-700">
              {field.label}
              {#if field.required}
                <span class="text-red-500">*</span>
              {/if}
            </label>

            {#if field.type === 'name'}
              <div class="grid grid-cols-2 gap-4">
                <input
                  type="text"
                  class="w-full p-2 border rounded-md"
                  placeholder="First name"
                  required={field.required}
                  bind:value={formResponses[`${field.id}_first`]}
                />
                <input
                  type="text"
                  class="w-full p-2 border rounded-md"
                  placeholder="Last name"
                  required={field.required}
                  bind:value={formResponses[`${field.id}_last`]}
                />
              </div>
            {:else if field.type === 'multipleChoice'}
              <div class="flex space-x-4">
                {#each field.options || [] as option, i}
                  <label class="inline-flex items-center">
                    <input
                      type="radio"
                      name={field.id}
                      value={option}
                      required={field.required}
                      bind:group={formResponses[field.id]}
                      class="mr-2"
                    />
                    <span>{option}</span>
                  </label>
                {/each}
              </div>
            {:else if field.type === 'checkbox'}
              <div class="flex space-x-4">
                {#each field.options || [] as option, i}
                  <label class="inline-flex items-center">
                    <input
                      type="checkbox"
                      value={option}
                      bind:group={formResponses[field.id]}
                      class="mr-2"
                    />
                    <span>{option}</span>
                  </label>
                {/each}
              </div>
            {:else if field.type === 'dropdown'}
              <select
                class="w-full p-2 border rounded-md"
                required={field.required}
                bind:value={formResponses[field.id]}
              >
                <option value="">Select an option</option>
                {#each field.options || [] as option}
                  <option value={option}>{option}</option>
                {/each}
              </select>
            {:else}
              <input
                type={field.type === 'shortText' ? 'text' :
                      field.type === 'email' ? 'email' :
                      field.type === 'phone' ? 'tel' :
                      field.type === 'number' ? 'number' :
                      field.type === 'date' ? 'date' :
                      field.type === 'time' ? 'time' :
                      field.type === 'file' ? 'file' :
                      field.type === 'price' ? 'number' : 'text'}
                class="w-full p-2 border rounded-md"
                required={field.required}
                bind:value={formResponses[field.id]}
                step={field.type === 'price' ? '0.01' : undefined}
                placeholder={field.description}
              />
            {/if}
          </div>
        {/each}

        <div class="flex justify-end space-x-4 mt-6">
          <button
            type="submit"
            class="px-4 py-2 bg-blue-500 text-white rounded-md hover:bg-blue-600 cursor-pointer"
          >
            Submit
          </button>
        </div>
      </form>
    {/if}

    <!-- Toggle Button -->
    <div class="mt-6 flex justify-end">
      <button
        class="cursor-pointer px-4 py-2 bg-green-500 text-white rounded-md hover:bg-green-600"
        on:click={togglePreview}
      >
        {isPreviewMode ? 'Back to Editor' : 'Preview Form'}
      </button>
    </div>
  </div>
</div>
  
<style>
  :global(.dndzone) {
    min-height: 60px;
  }
</style>
