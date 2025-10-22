<script setup lang="ts">
const files = ref<File[]>([]);

//agregamos una lista de mimetypes permitidos
//en la documentacion viene como podemos manejar el atributo que mandamos en accept
//podemos usar asteriscos como wildcards para definir un grupo de archivos
//o tambien podemos usar en este caso este codigo que nos permite aceptar archivos de word
const allowedFileTypes = ['image/*','.doc,.docx,.xml,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document'];

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
			<input type="file" :accept="allowedFileTypes.join(',')" multiple @change="handleFileSelect" hidden />
			Upload
		</label>
		<p v-for="file in files" :key="file.name">
			{{ file.name
			}}
			<!-- si el archivo que vamos a mostrar no está en los mimetypes permitidos
			mostramos un mensaje de error -->
			<!-- <span v-if="!allowedFileTypes.includes(file.type)" class="text-red-500">
				File type not allowed
			</span> -->
		</p>
	</div>
</template>

