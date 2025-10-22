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

//creamos esta propiedad computada, para convertir en tiempo
//real los archivos de imagen en urls que podemos usar y previsualizarlas
const previews = computed(() => {
	return files.value.map((file) => {
		//si el archivo es de tipo imagen, retornamos
		//la url con la vista previa, usando esta funcion nativa de js
		//modificamos la funcion para que genere los urls de videos e imagenes
		if (file.type.startsWith('image/')) {
			return{
				//retornamos un objeto para agregar
				//metadata del tipo de archivo
				//para manejarlo correctamente al mostrarlo
				type: 'image',
				url: URL.createObjectURL(file),
			}
			return URL.createObjectURL(file);
		} else if(file.type.startsWith('video/')){
			return{
				type: 'video',
				url: URL.createObjectURL(file),
			}
		} else{ 
			//sino retornamos null para no mostrarlas
			return null;
		}
	});
});

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
			 agregamos autoplay para que se reprodusca automaticamente, loop para que se repita y controls para mostrar controles de reproduccionque el usuario pueda usar

			<video v-if="previews[index]?.type === 'video'" autoplay loop controls :src="previews[index]?.url" >

			</video>

			<!-- imprimos este mensaje de error si el archivo es muy grande -->

			<span v-if="fileIsTooBig(file)" class="text-red-500"
				>File is too big. File must be no larger than
				{{ maxMB }}MB</span
			>
		</p>
	</div>
</template>

