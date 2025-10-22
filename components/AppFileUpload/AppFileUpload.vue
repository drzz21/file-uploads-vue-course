<script setup lang="ts">
const files = ref<File[]>([]);

//agregamos una lista de mimetypes permitidos
//en la documentacion viene como podemos manejar el atributo que mandamos en accept
//podemos usar asteriscos como wildcards para definir un grupo de archivos
//o tambien podemos usar en este caso este codigo que nos permite aceptar archivos de word
const allowedFileTypes = [
	'image/*',
	'video/*',
	'.doc,.docx,.xml,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document',
];

//vamos ahora a definir tamaños máximos para los archivos
//definimos una variable para manejar en megas el tamaño de archivos
const maxMB = 5;
//multiplicamos por 1024*1024 para tener el tamaño en kilobytes y luego en megabytes
const MAX_FILE_SIZE = maxMB * 1024 * 1024; // 5MB

//definimos una funcion que valide si el archivo es muy grande
//si pasa el tamaño maximo retornamos true
function fileIsTooBig(file: File): boolean {
	return file.size > MAX_FILE_SIZE;
}

function handleFileSelect(event: Event) {
	//accedemos al elemento que disparó el evento, indicamos a ts que es un htmlInputElement
	//con esto podemos acceder a la propiedad files
	const input = event.target as HTMLInputElement;
	//input files es un objeto que contiene los archivos seleccionados
	//lo convertimos a un array con Array.from
	//si por algo es undefined, le pasamos un array vacío para evitar errores
	const fileAsArray = Array.from(input?.files || []); // FileList
	//concatenamos los archivos seleccionados a nuestra referencia reactiva
	files.value = files.value.concat(fileAsArray);
	// console.log(event);
}
</script>

<template>
	<div>
		<!-- follow along here -->
		<!-- input oculto con atributo multiple para seleccionar varios archivos -->
		<label class="px-4 py-2 bg-blue-500 text-white rounded cursor-pointer">
			<!-- usamos accept que es la forma descrita en la documentacion para listar el tipo de archivos a aceptar -->
			<input
				type="file"
				:accept="allowedFileTypes.join(',')"
				multiple
				@change="handleFileSelect"
				hidden
			/>
			Upload
		</label>
		<p v-for="file in files" :key="file.name">
			{{ file.name }}
			<!-- si el archivo que vamos a mostrar no está en los mimetypes permitidos
			mostramos un mensaje de error -->
			<!-- <span v-if="!allowedFileTypes.includes(file.type)" class="text-red-500">
				File type not allowed
			</span> -->

			<!-- imprimos este mensaje de error si el archivo es muy grande -->

			<span v-if="fileIsTooBig(file)" class="text-red-500"
				>File is too big. File must be no larger than
				{{ maxMB }}MB</span
			>
		</p>
	</div>
</template>

