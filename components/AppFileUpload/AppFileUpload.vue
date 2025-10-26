<!-- se mueve a nuevo script porque las props del setup no pueden hacer referencia 
la constante si está dentro del mismo setup, hay que moverlo a otro script sin setup -->

<script lang="ts">
// se importa attrAccept para la validacion de los archivos
//seleccionados en el file input
import attrAccept from 'attr-accept';

const allowedFileTypes = [
	'image/*',
	'video/*',
	'.doc,.docx,.xml,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document',
];
</script>

<script setup lang="ts">
const files = ref<File[]>([]);

// se crea variable auxiliar para manejar el estado de drag and drop
// cuando se esté arrastrando un archivo sobre el area del input
// se usará para cambiar estilos
const isDragging = ref(false);

// agregamos las props con valores por defecto para las siguientes variables usadas en nuestro input

const props = withDefaults(
	defineProps<{
		accept?: string[];
		maxMb?: number;
		multiple?: boolean;
	}>(),
	{
		accept: () => allowedFileTypes,
		maxMb: 5,
		multiple: false,
	}
);

//agregamos una lista de mimetypes permitidos
//en la documentacion viene como podemos manejar el atributo que mandamos en accept
//podemos usar asteriscos como wildcards para definir un grupo de archivos
//o tambien podemos usar en este caso este codigo que nos permite aceptar archivos de word

//agregamos este emit para enviar al padre las propiedades de fileName y url
//de los archivos subidos
//agregamos aquí el tipado

const emit = defineEmits<{
	'uploaded:files': [
		files: {
			filename: string;
			url: string;
		}[]
	];
}>();

//vamos ahora a definir tamaños máximos para los archivos
//definimos una variable para manejar en megas el tamaño de archivos
// const maxMB = 5;
//multiplicamos por 1024*1024 para tener el tamaño en kilobytes y luego en megabytes
// const MAX_FILE_SIZE = maxMB * 1024 * 1024; // 5MB

//definimos una funcion que valide si el archivo es muy grande
//si pasa el tamaño maximo retornamos true
//ya no tenemos nuestra constante entonces tomamos los mb de la prop
//y con esa prop hacemos el calculo para convertir a megas
function fileIsTooBig(file: File): boolean {
	// return file.size > MAX_FILE_SIZE;
	return file.size > props.maxMb * 1024 * 1024;
}

//agregamos este tipo para poder manejarlo en las computed properties
//para que la propertie sepa el valor que retorna
type Preview = {
	type: string;
	url: string;
} | null;
//creamos esta propiedad computada, para convertir en tiempo
//real los archivos de imagen en urls que podemos usar y previsualizarlas
//agregamos el tipado
const previews = computed<Preview[]>((oldPreviews) => {
	return files.value.map((file) => {
		//revisamos nuestro valor anterior cuando cambie la propertie
		//y revocamos las url para evitar problemas de memoria
		if (oldPreviews) {
			oldPreviews.map((oldPreview) => {
				if (oldPreview?.url) {
					URL.revokeObjectURL(oldPreview.url);
				}
			});
		}

		//si el archivo es de tipo imagen, retornamos
		//la url con la vista previa, usando esta funcion nativa de js
		//modificamos la funcion para que genere los urls de videos e imagenes
		if (file.type.startsWith('image/')) {
			return {
				//retornamos un objeto para agregar
				//metadata del tipo de archivo
				//para manejarlo correctamente al mostrarlo
				type: 'image',
				url: URL.createObjectURL(file),
			};
			// return URL.createObjectURL(file);
		} else if (file.type.startsWith('video/')) {
			return {
				type: 'video',
				url: URL.createObjectURL(file),
			};
		} else {
			//sino retornamos null para no mostrarlas
			return null;
		}
	});
});

//al desmontar el componente de igual forma liberamos las url
//para evitar problemas de memoria
onUnmounted(() => {
	//liberamos la memoria de los object urls creados
	previews.value.forEach((preview) => {
		if (preview?.url) {
			URL.revokeObjectURL(preview.url);
		}
	});
});

async function handleFileSelect(event: Event) {
	//accedemos al elemento que disparó el evento, indicamos a ts que es un htmlInputElement
	//con esto podemos acceder a la propiedad files
	const input = event.target as HTMLInputElement;
	//input files es un objeto que contiene los archivos seleccionados
	//lo convertimos a un array con Array.from
	//si por algo es undefined, le pasamos un array vacío para evitar errores
	const fileAsArray = Array.from(input?.files || []); // FileList

	//lo recomendado para subir archivos
	//porque automaticamente agrega el multipart/form-data
	const formData = new FormData();
	//agregamos los archivos al formdata

	fileAsArray.map((file) => {
		//solo se agragrán al form data los archivos
		//validos en formato y tamaño
		if (attrAccept(file, props.accept) && !fileIsTooBig(file)) {
			formData.append(file.name, file);
		}
	});

	//si hay al menos un archivo en el formdata hacemos el fetch
	if (Array.from(formData.values()).length) {
		const response = await fetch('/api/upload', {
			method: 'POST',
			body: formData,
		});

		const data = (await response.json()) as {
			files: { filename: string; url: string }[];
		};

		emit('uploaded:files', data.files);
	}

	//hacemos el fetch a nuestra api de nuxt,
	//enviando el formdata como body
	//y usando el metodo post

	//se usa $fetch porque es la forma recomendada en nuxt
	//para hacer peticiones http, maneja automaticamente
	//la conversion a json y otros detalles

	//usaremos la conversiona  json del fetch normal para ilustrar el ejemplo
	//para este caso especifico

	//emitimos el evento con los archivos subidos

	//respondemos la respuesta

	// console.log(response);

	// manejamos la referencia reactiva de archivos
	//si son archivos multiples concatenamos los archivos cargados
	//para que se puedan agregar varios, si no son multiples simplemente
	//reemplazamos la variable con el nuevo valor

	//este codigo queda fuera del if porque es el que nos sirve
	//para previsualizar los archivos, es parecido a un optimistic update
	if (props.multiple) {
		files.value = files.value.concat(fileAsArray);
	} else {
		files.value = fileAsArray;
	}

	//concatenamos los archivos seleccionados a nuestra referencia reactiva

	// console.log(event);
}
</script>

<template>
	<div class="@container w-full">
		<!-- follow along here -->
		<!-- input oculto con atributo multiple para seleccionar varios archivos -->
		<!-- <label class="px-4 py-2 bg-blue-500 text-white rounded cursor-pointer"> -->
		<!-- usamos accept que es la forma descrita en la documentacion para listar el tipo de archivos a aceptar -->
		<!-- agregamos las referencias a las props -->
		<!-- quitamos nuestro label porque usaremos un div ahora -->
		<!-- con estos estilos para posicionar el input dentro de ese div -->
		<!-- con inset 0 tenemos top 0, right 0, bottom 0 y left 0 -->
		<!-- agregamos estilos condicionales para cuando se esté arrastrando un archivo sobre el area del input -->
		<!-- manejamos la inicializacion de la variable isDragging en true o false
		  dependiendo del estatus del drag and drop  -->
		<div
			class="relative border-2 border-dashed rounded-xl p-8 @md:p-12 text-center transition-all duration-300 ease-in-out cursor-pointer hover:border-blue-400 hover:bg-blue-50/50 group"
			:class="{
				'border-blue-500 bg-blue-50 shadow-lg scale-[1.02]': isDragging,
				'border-gray-300 bg-gray-50/50': !isDragging,
			}"
			@dragenter="($event) => (isDragging = true)"
			@dragover="($event) => (isDragging = true)"
			@dragleave="($event) => (isDragging = false)"
			@drop="($event) => (isDragging = false)"
		>
			<input
				class="absolute inset-0 opacity-0 cursor-pointer"
				type="file"
				:accept="accept?.join(',')"
				:multiple="multiple"
				@change="handleFileSelect"
			>

			<!-- esto se agregó con ia usando este prompt -->
			<!-- add a file upload svg -->

			<svg
				class="w-12 h-12 @md:w-16 @md:h-16 mx-auto mb-4 transition-all duration-300"
				:class="isDragging ? 'text-blue-500 scale-110' : 'text-gray-400 group-hover:text-blue-500'"
				fill="none"
				stroke="currentColor"
				viewBox="0 0 24 24"
				xmlns="http://www.w3.org/2000/svg"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
				/>
			</svg>

			<!-- añadimos texto para nuestro upload -->
			<div class="space-y-2">
				<p class="text-base @md:text-lg font-medium">
					<span class="text-blue-600 hover:text-blue-700 transition-colors">Upload files</span>
					<span class="text-gray-600"> or drag and drop</span>
				</p>
				<p class="text-xs @md:text-sm text-gray-500">
					{{ accept?.join(', ') }} (max {{ maxMb }}MB)
				</p>
			</div>
		</div>
		<!-- Upload -->
		<!-- </label> -->
		
		<!-- Files Preview Section -->
		<div v-if="files.length > 0" class="mt-4 space-y-3">
			<div 
				v-for="(file, index) in files" 
				:key="file.name"
				class="bg-white rounded-lg border border-gray-200 hover:border-gray-300 transition-colors duration-200"
			>
				<div class="p-3 @md:p-4">
					<!-- Horizontal Layout: Thumbnail + Info + Status + Close -->
					<div class="flex items-center gap-3">
						<!-- Thumbnail -->
						<div class="flex-shrink-0">
							<!-- Image Preview -->
							<img
								v-if="previews[index]?.type === 'image'"
								:src="previews[index]?.url"
								:alt="file.name"
								class="w-12 h-12 rounded-lg object-cover border border-gray-200"
							>
							
							<!-- Video Preview -->
							<div v-else-if="previews[index]?.type === 'video'" class="w-12 h-12 rounded-lg overflow-hidden border border-gray-200">
								<video
									:src="previews[index]?.url"
									class="w-full h-full object-cover"
								/>
							</div>
							
							<!-- Default File Icon -->
							<div v-else class="w-12 h-12 bg-gray-100 rounded-lg flex items-center justify-center border border-gray-200">
								<svg class="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
									<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21h10a2 2 0 002-2V9.414a1 1 0 00-.293-.707l-5.414-5.414A1 1 0 0012.586 3H7a2 2 0 00-2 2v14a2 2 0 002 2z" />
								</svg>
							</div>
						</div>

						<!-- File Info -->
						<div class="flex-1 min-w-0">
							<p class="text-sm font-medium text-gray-900 truncate">
								{{ file.name }}
							</p>
							<p class="text-xs text-gray-500 mt-0.5">
								{{ (file.size / (1024 * 1024)).toFixed(2) }} MB
							</p>
						</div>

						<!-- Status or Error -->
						<div class="flex-shrink-0">
							<!-- Error State -->
							<div v-if="fileIsTooBig(file) || !attrAccept(file, accept)" class="text-red-500">
								<svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
									<path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
								</svg>
							</div>
							
							<!-- Success State -->
							<div v-else class="flex items-center gap-2">
								<svg class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
									<path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
								</svg>
								<span class="text-xs text-green-600 font-medium">Done</span>
							</div>
						</div>

						<!-- Close Button -->
						<button 
							type="button"
							class="flex-shrink-0 text-gray-400 hover:text-gray-600 transition-colors"
							@click="files.splice(index, 1)"
						>
							<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
							</svg>
						</button>
					</div>

					<!-- Error Messages (Below the main row) -->
					<div v-if="fileIsTooBig(file) || !attrAccept(file, accept)" class="mt-2 ml-15">
						<p v-if="fileIsTooBig(file)" class="text-xs text-red-600">
							File is too big. Must be no larger than {{ maxMb }}MB
						</p>
						<p v-if="!attrAccept(file, accept)" class="text-xs text-red-600">
							File type not accepted. Acceptable: {{ accept?.join(', ') }}
						</p>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

