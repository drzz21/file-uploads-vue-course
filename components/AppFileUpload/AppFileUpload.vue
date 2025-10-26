<!-- se mueve a nuevo script porque las props del setup no pueden hacer referencia 
la constante si está dentro del mismo setup, hay que moverlo a otro script sin setup -->

<script lang="ts">
const allowedFileTypes = [
	'image/*',
	'video/*',
	'.doc,.docx,.xml,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document',
];
</script>

<script setup lang="ts">
const files = ref<File[]>([]);

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
		formData.append(file.name, file);
	});

	//hacemos el fetch a nuestra api de nuxt,
	//enviando el formdata como body
	//y usando el metodo post

	const response = await fetch('/api/upload', {
		method: 'POST',
		body: formData,
	});

	//se usa $fetch porque es la forma recomendada en nuxt
	//para hacer peticiones http, maneja automaticamente
	//la conversion a json y otros detalles

	//usaremos la conversiona  json del fetch normal para ilustrar el ejemplo
	//para este caso especifico

	const data = (await response.json()) as {
		files: { filename: string; url: string }[];
	};

	//emitimos el evento con los archivos subidos

	emit('uploaded:files', data.files);

	//respondemos la respuesta

	console.log(response);

	// manejamos la referencia reactiva de archivos
	//si son archivos multiples concatenamos los archivos cargados
	//para que se puedan agregar varios, si no son multiples simplemente
	//reemplazamos la variable con el nuevo valor

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
	<div>
		<!-- follow along here -->
		<!-- input oculto con atributo multiple para seleccionar varios archivos -->
		<!-- <label class="px-4 py-2 bg-blue-500 text-white rounded cursor-pointer"> -->
		<!-- usamos accept que es la forma descrita en la documentacion para listar el tipo de archivos a aceptar -->
		<!-- agregamos las referencias a las props -->
		<!-- quitamos nuestro label porque usaremos un div ahora -->
		<!-- con estos estilos para posicionar el input dentro de ese div -->
		<!-- con inset 0 tenemos top 0, right 0, bottom 0 y left 0 -->
		<div class="relative border border-dashed p-5 rounded text-center">
			<input
				class="absolute inset-0 opacity-0"
				type="file"
				:accept="accept?.join(',')"
				:multiple="multiple"
				@change="handleFileSelect"
			/>

			<!-- esto se agregó con ia usando este prompt -->
			<!-- add a file upload svg -->

			<svg
				class="w-12 h-12 mx-auto mb-4 text-gray-400"
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
				></path>
			</svg>

			<!-- añadimos texto para nuestro upload -->
			<span class="text-blue-500">Upload files</span>
			<span> or drag an drop</span>
		</div>
		<!-- Upload -->
		<!-- </label> -->
		<p v-for="(file, index) in files" :key="file.name">
			{{ file.name }}
			<!-- si el archivo que vamos a mostrar no está en los mimetypes permitidos
			mostramos un mensaje de error -->
			<!-- <span v-if="!allowedFileTypes.includes(file.type)" class="text-red-500">
				File type not allowed
			</span> -->

			<!-- vamos a previsualizar nuestra imagen, usando, la propiedad computada
			 y el indice de nuestro v-for enviandolo a dicha propiedad computada -->
			<!-- modificamos para agregar los nuevos atributos de tipo y url -->
			<img
				v-if="previews[index]?.type === 'image'"
				:src="previews[index]?.url"
				:alt="file.name"
			/>

			<!-- agregamos el tag de video para previsualizar videos -->
			<!-- agregamos autoplay para que se reprodusca automaticamente, loop para que se repita y controls para mostrar controles de reproduccionque el usuario pueda usar -->

			<video
				v-if="previews[index]?.type === 'video'"
				autoplay
				loop
				controls
				:src="previews[index]?.url"
			></video>

			<!-- imprimos este mensaje de error si el archivo es muy grande -->

			<span v-if="fileIsTooBig(file)" class="text-red-500"
				>File is too big. File must be no larger than
				{{ maxMb }}MB</span
			>
		</p>
	</div>
</template>

