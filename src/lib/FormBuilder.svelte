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
  
    let fieldTypes: { type: FieldType; label: string; icon: string }[] = [
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
      { type: 'region', label: 'Region & City', icon: 'material-symbols:location-on' },
    ];
  
    interface Region {
      id: string;
      name: string;
      code: any;
    }

    interface City {
      id: string;
      name: string;
      code: any;
    }

    let regions: Region[] = [];
    let cities: Record<string, City[]> = {};
    let selectedRegion = '';

    async function fetchRegions() {
      try {
        const response = await fetch('https://psgc.gitlab.io/api/regions/');
        regions = await response.json();
      } catch (error) {
        console.error('Error fetching regions:', error);
      }
    }

    async function fetchCities(regionCode: string) {
    try {
        const response = await fetch(`https://psgc.gitlab.io/api/regions/${regionCode}/cities-municipalities/`);
        cities[regionCode] = await response.json();
        console.log("Fetched cities:", cities[regionCode]);
    } catch (error) {
        console.error('Error fetching cities:', error);
    }
}



    function addField(type: FieldType) {
      if (type === 'region') {
        formData.fields = [...formData.fields, 
          {
            id: crypto.randomUUID(),
            type: 'region',
            label: 'Region',
            required: true,
            options: regions.map(region => region.name)
          },
          {
            id: crypto.randomUUID(),
            type: 'city',
            label: 'City/Municipality',
            required: true,
            options: []
          },
          {
            id: crypto.randomUUID(),
            type: 'shortText',
            label: 'Street/Barangay',
            required: true
          }
        ];
        
        fieldTypes = fieldTypes.filter(ft => ft.type !== 'region');
      } else {
        const newField: FormFieldTypes = {
          id: crypto.randomUUID(),
          type,
          label: `New ${type} field`,
          required: false,
          options: type === 'multipleChoice' || type === 'checkbox' || type === 'dropdown' ? ['Option 1'] : undefined
        };
        formData.fields = [...formData.fields, newField];
      }
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

    import { onMount } from 'svelte';
    onMount(() => {
      fetchRegions();
    });

    async function handleRegionChange(event: Event, fieldId: string) {
    const regionName = (event.target as HTMLSelectElement).value;
    const region = regions.find(r => r.name === regionName);
    if (region) {
        console.log(region)
        await fetchCities(region.code);
        formData.fields = formData.fields.map(f => {
            if (f.type === 'city') {
                return {
                    ...f,
                    options: cities[region.code]?.map(city => city.name) || []
                };
            }
            return f;
        });

        console.log("Updated form fields:", formData.fields);
    }
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
            <label 
              for={field.id} 
              class="block text-sm font-medium text-gray-700"
            >
              {field.label}
              {#if field.required}
                <span class="text-red-500">*</span>
              {/if}
            </label>

            {#if field.type === 'name'}
              <div class="grid grid-cols-2 gap-4">
                <input
                  type="text"
                  id={`${field.id}_first`}
                  class="w-full p-2 border rounded-md cursor-pointer"
                  placeholder="First name"
                  required={field.required}
                  bind:value={formResponses[`${field.id}_first`]}
                />
                <input
                  type="text"
                  id={`${field.id}_last`}
                  class="w-full p-2 border rounded-md cursor-pointer"
                  placeholder="Last name"
                  required={field.required}
                  bind:value={formResponses[`${field.id}_last`]}
                />
              </div>
            {:else if field.type === 'phone'}
              <div class="relative">
                <span class="absolute left-3 top-2">+63</span>
                <input
                  type="tel"
                  id={field.id}
                  class="w-full p-2 pl-12 border rounded-md cursor-pointer"
                  pattern="[0-9]{10}"
                  placeholder="9XX XXX XXXX"
                  required={field.required}
                  bind:value={formResponses[field.id]}
                />
              </div>
            {:else if field.type === 'price'}
              <div class="relative">
                <span class="absolute left-3 top-2">₱</span>
                <input
                  type="number"
                  id={field.id}
                  class="w-full p-2 pl-8 border rounded-md cursor-pointer"
                  placeholder="0.00"
                  step="0.01"
                  required={field.required}
                  bind:value={formResponses[field.id]}
                />
              </div>
            {:else if field.type === 'multipleChoice'}
              <div class="flex flex-wrap gap-4">
                {#each field.options || [] as option, i}
                  <label class="inline-flex items-center cursor-pointer">
                    <input
                      type="radio"
                      id={`${field.id}_${i}`}
                      name={field.id}
                      value={option}
                      required={field.required}
                      bind:group={formResponses[field.id]}
                      class="mr-2 cursor-pointer"
                    />
                    <span>{option}</span>
                  </label>
                {/each}
              </div>
            {:else if field.type === 'checkbox'}
              <div class="flex flex-wrap gap-4">
                {#each field.options || [] as option, i}
                  <label class="inline-flex items-center cursor-pointer">
                    <input
                      type="checkbox"
                      id={`${field.id}_${i}`}
                      value={option}
                      bind:group={formResponses[field.id]}
                      class="mr-2 cursor-pointer"
                    />
                    <span>{option}</span>
                  </label>
                {/each}
              </div>
            {:else if field.type === 'region'}
              <select
                id={field.id}
                class="w-full p-2 border rounded-md cursor-pointer"
                required={field.required}
                bind:value={formResponses[field.id]}
                on:change={(e) => handleRegionChange(e, field.id)}
              >
                <option value="">Select Region</option>
                {#each regions as region}
                  <option value={region.name}>{region.name}</option>
                {/each}
              </select>
            {:else if field.type === 'city'}
              <select
                id={field.id}
                class="w-full p-2 border rounded-md cursor-pointer"
                required={field.required}
                bind:value={formResponses[field.id]}
              >
                <option value="">Select City/Municipality</option>
                {#each field.options || [] as city}
                  <option value={city}>{city}</option>
                {/each}
              </select>
            {:else}
              <input
                id={field.id}
                type={field.type === 'shortText' ? 'text' :
                      field.type === 'email' ? 'email' :
                      field.type === 'number' ? 'number' :
                      field.type === 'date' ? 'date' :
                      field.type === 'time' ? 'time' :
                      field.type === 'file' ? 'file' : 'text'}
                class="w-full p-2 border rounded-md cursor-pointer"
                required={field.required}
                bind:value={formResponses[field.id]}
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
