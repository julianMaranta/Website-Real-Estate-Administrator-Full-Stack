<template>
  <div class="container">
    <A_Sidebar />
    <div class="content">
      <div class="logo-container">
        <img src="@/assets/logo-real-state.png" alt="Logo Inmobiliaria" class="logo-plataforma" />
      </div>
      <br>   
      <br>

      <!-- Mostrar mensaje de error si existe -->
      <div v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </div>

      <div v-if="loading" class="loading">Cargando propiedades...</div>
      
      <!-- Mensaje cuando no hay propiedades -->
      <div v-else-if="propiedades.length === 0" class="no-properties">
        <h3>No hay propiedades en alquiler disponibles</h3>
        <p>No se encontraron propiedades para editar.</p>
      </div>
      
      <div v-else>
        <!-- Listado de propiedades -->
        <div v-for="propiedad in propiedades" :key="propiedad.id" class="property-list-item">
          <div class="property-item" @click="selectProperty(propiedad.id)">
            <h3>{{ propiedad.direccion }}</h3>
            <div class="property-details">
              <span><strong>Tipo:</strong> {{ propiedad.tipoPropiedad }}</span>
              <span><strong>Precio:</strong> ${{ propiedad.precioAlquiler?.toLocaleString() }} {{ propiedad.moneda }}</span>
              <span><strong>Ubicación:</strong> {{ propiedad.ubicacion }}</span>
            </div>
            <div class="property-status">
              <span :class="`status-badge ${propiedad.estado?.toLowerCase() || 'disponible'}`">
                {{ propiedad.estado || 'Disponible' }}
              </span>
            </div>
          </div>
          
          <!-- Formulario de edición -->
          <div v-if="selectedPropertyId === propiedad.id" class="property-edit-form">
            <h2>Editar Propiedad en Alquiler</h2>
            <form @submit.prevent="showModal = true" class="edit-form">
              <div class="form-row">
                <div class="form-group">
                  <label for="direccion">Dirección</label>
                  <input id="direccion" v-model="formData.direccion" required />
                </div>

                <div class="form-group">
                  <label for="ubicacion">Ubicación</label>
                  <select id="ubicacion" v-model="formData.ubicacion" required>
                    <option value="">Seleccione una ubicación</option>
                    <optgroup label="Zona Centro">
                      <option value="La Plata (Casco Urbano)">La Plata (Casco Urbano)</option>
                      <option value="Tolosa">Tolosa</option>
                      <option value="Villa Elvira">Villa Elvira</option>
                    </optgroup>
                    
                    <optgroup label="Barrios Residenciales">
                      <option value="City Bell">City Bell</option>
                      <option value="Manuel B. Gonnet">Manuel B. Gonnet</option>
                      <option value="Villa Elisa">Villa Elisa</option>
                      <option value="Ensenada">Ensenada</option>
                      <option value="Los Hornos">Los Hornos</option>
                    </optgroup>
                    
                    <optgroup label="Localidades Periféricas">
                      <option value="Berazategui">Berazategui</option>
                      <option value="Abasto">Abasto</option>
                      <option value="Joaquín Gorina">Joaquín Gorina</option>
                      <option value="Lisandro Olmos">Lisandro Olmos</option>
                      <option value="Melchor Romero">Melchor Romero</option>
                      <option value="Ringuelet">Ringuelet</option>
                    </optgroup>
                    
                    <option value="Mar del Plata">Mar del Plata</option>
                  </select>
                </div>
              </div>

              <div class="form-row">
                <div class="form-group">
                  <label for="tipo-propiedad">Tipo de Propiedad</label>
                  <select id="tipo-propiedad" v-model="formData.tipoPropiedad" required>
                    <option value="">Seleccione un tipo</option>
                    <option value="Casa">Casa</option>
                    <option value="Monoambiente">Monoambiente</option>
                    <option value="Departamento">Departamento</option>
                    <option value="PH">PH</option>
                    <option value="Local Comercial">Local Comercial</option>
                    <option value="Oficina">Oficina</option>
                    <option value="Quinta">Quinta</option>
                    <option value="Cochera">Cochera</option>
                    <option value="Hotel">Hotel</option>
                    <option value="Terreno">Terreno</option>
                    <option value="Campo">Campo</option>
                    <option value="Fondo de Comercio">Fondo de Comercio</option>
                    <option value="Galpón">Galpón</option>
                  </select>
                </div>

                <div class="form-group">
                  <label>Precio Alquiler</label>
                  <div class="price-container">
                    <input 
                      id="precio-alquiler" 
                      v-model.number="formData.precioAlquiler" 
                      type="number" 
                      min="0"
                      :placeholder="formData.moneda === 'ARS' ? 'Ej: 150000' : 'Ej: 150'"
                      class="price-input"
                      required
                    />
                    <div class="currency-options">
                      <label class="currency-option">
                        <input 
                          type="radio" 
                          v-model="formData.moneda" 
                          value="ARS" 
                        />
                        ARS
                      </label>
                      <label class="currency-option">
                        <input 
                          type="radio" 
                          v-model="formData.moneda" 
                          value="USD" 
                        />
                        USD
                      </label>
                    </div>
                  </div>
                </div>
              </div>

              <div class="form-row">
                <div class="form-group">
                  <label for="precio-expensas" class="expensas-label">
                    Precio Expensas 
                    <span class="expensas-badge">ARS</span>
                  </label>
                  <input 
                    id="precio-expensas" 
                    v-model.number="formData.precioExpensas" 
                    type="number" 
                    min="0" 
                    placeholder="Ej: 15000"
                  />
                  <div class="checkbox-option">
                    <input type="checkbox" id="sin-expensas" v-model="sinExpensas" @change="handleExpensasChange">
                    <label for="sin-expensas">Sin Expensas</label>
                  </div>
                </div>

                <div class="form-group">
                  <label for="metros-cuadrados-totales">Metros Cuadrados Totales (m²)</label>
                  <input id="metros-cuadrados-totales" v-model.number="formData.metrosCuadradosTotales" type="number" placeholder="Ej: 80" />
                </div>
              </div>

              <div class="form-group">
                <label for="metros-cuadrados-cubiertos">Metros Cuadrados Cubiertos (m²)</label>
                <input id="metros-cuadrados-cubiertos" v-model.number="formData.metrosCuadradosCubiertos" type="number" placeholder="Ej: 80" />
              </div>

              <!-- Sección de características -->
              <div class="form-section-title">Características</div>

              <div class="form-row">
                <div class="form-group">
                  <label for="habitaciones">Habitaciones</label>
                  <input id="habitaciones" v-model.number="formData.habitaciones" type="number" min="0" />
                </div>

                <div class="form-group">
                  <label for="banos">Baños</label>
                  <input id="banos" v-model.number="formData.banos" type="number" min="0" />
                </div>
              </div>

              <div class="form-row">
                <div class="form-group">
                  <label for="cochera">Cochera</label>
                  <select id="cochera" v-model="formData.cochera">
                    <option value="Si">Sí</option>
                    <option value="No">No</option>
                  </select>
                </div>
              </div>

              <!-- Sección de información adicional -->
              <div class="form-section-title">Información Adicional</div>

              <div class="form-group">
                <label for="descripcion">Descripción</label>
                <textarea id="descripcion" v-model="formData.descripcion" rows="4"></textarea>
              </div>

              <div class="form-group">
                <label for="map-link">Link Google Maps</label>
                <input id="map-link" v-model="formData.mapLink" placeholder="URL de Google Maps" />
              </div>

              <div class="form-group">
                <label for="youtube-video-url">Link del Video de YouTube</label>
                <input v-model="formData.youtubeVideoUrl" id="youtube-video-url" placeholder="Link del video de YouTube" />
              </div>

              <!-- Área de imágenes con drag & drop -->
              <div class="form-section-title">Gestión de Imágenes</div>
              
              <div class="form-group">
                <label for="imagenes">Imágenes de la Propiedad</label>
                
                <!-- Área de Drag and Drop para subir imágenes -->
                <div 
                  class="dropzone"
                  :class="{ 'dragover': isDragover, 'has-images': uploadedImages.length > 0 }"
                  @dragover.prevent="handleDragOver"
                  @dragleave.prevent="handleDragLeave"
                  @drop.prevent="handleDrop"
                  @click="handleDropzoneClick"
                  ref="dropzoneElement"
                >
                  <div v-if="uploadedImages.length === 0" class="dropzone-empty">
                    <div class="dropzone-icon">
                      📤
                    </div>
                    <p>Arrastra y suelta imágenes aquí</p>
                    <p class="dropzone-subtext">o haz clic para seleccionar archivos</p>
                    <p class="dropzone-hint">Formatos soportados: JPG, PNG, GIF (Máx. 10MB)</p>
                  </div>
                  
                  <div v-else class="dropzone-preview">
                    <!-- Instrucciones de reordenamiento -->
                    <div class="reorder-instructions">
                      <div class="instructions-content">
                        <span class="reorder-hint">📌 Arrastra y suelta las imágenes para reordenarlas</span>
                        <span class="image-count">{{ uploadedImages.length }}/20 imágenes</span>
                      </div>
                      <button 
                        type="button" 
                        @click.stop="triggerFileInput" 
                        class="add-more-button"
                        :disabled="uploadedImages.length >= 20"
                      >
                        + Agregar más
                      </button>
                    </div>
                    
                    <!-- Lista de imágenes reordenables -->
                    <div 
                      class="images-grid"
                      @dragover.prevent
                      @drop.prevent="handleGridDrop"
                    >
                      <div 
                        v-for="(image, index) in uploadedImages" 
                        :key="image.id"
                        class="image-preview-container"
                        :class="{ 
                          'dragging': draggingImageId === image.id,
                          'drag-over': dragOverImageId === image.id,
                          'is-main': index === 0
                        }"
                        draggable="true"
                        @dragstart="handleImageDragStart($event, image)"
                        @dragover.prevent="handleImageDragOver($event, image)"
                        @dragenter.prevent="handleImageDragEnter($event, image)"
                        @dragleave="handleImageDragLeave"
                        @drop="handleImageDrop($event, image)"
                        @dragend="handleImageDragEnd"
                      >
                        <!-- Número de orden -->
                        <div class="image-order">
                          {{ index + 1 }}
                        </div>
                        
                        <!-- Indicador de imagen principal -->
                        <div class="main-indicator" v-if="index === 0">
                          ⭐ Principal
                        </div>
                        
                        <div class="image-preview">
                          <img 
                            :src="image.previewUrl || image.dropboxUrl || 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTAwIiBoZWlnaHQ9IjEwMCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxyZWN0IHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiBmaWxsPSIjZWVlZWVlIi8+PHRleHQgeD0iNTAiIHk9IjUwIiBmb250LWZhbWlseT0iQXJpYWwiIGZvbnQtc2l6ZT0iMTIiIHRleHQtYW5jaG9yPSJtaWRkbGUiIGR5PSIuM2VtIiBmaWxsPSIjOTk5Ij5JbWFnZW48L3RleHQ+PC9zdmc+'"
                            :alt="image.name || 'Imagen propiedad'"
                            class="preview-image"
                          />
                          <div v-if="image.uploading" class="upload-progress">
                            <div class="progress-bar">
                              <div class="progress-fill" :style="{ width: image.progress + '%' }"></div>
                            </div>
                            <span class="progress-text">{{ image.progress }}%</span>
                          </div>
                          <div v-else-if="image.error" class="upload-error">
                            ❌ Error
                          </div>
                          <div v-else-if="image.uploaded" class="upload-success">
                            ✅ Subido
                          </div>
                          
                          <!-- Indicador de arrastre -->
                          <div class="drag-indicator" title="Arrastrar para reordenar">
                            ⋮⋮
                          </div>
                          
                          <!-- Controles de orden -->
                          <div class="image-order-controls">
                            <button 
                              type="button" 
                              @click.stop="moveImageUp(index)"
                              :disabled="index === 0"
                              class="order-btn up-btn"
                              title="Mover arriba"
                            >
                              ↑
                            </button>
                            <button 
                              type="button" 
                              @click.stop="moveImageDown(index)"
                              :disabled="index === uploadedImages.length - 1"
                              class="order-btn down-btn"
                              title="Mover abajo"
                            >
                              ↓
                            </button>
                            <button 
                              type="button" 
                              @click.stop="setAsMainImage(index)"
                              :disabled="index === 0"
                              class="order-btn main-btn"
                              title="Establecer como principal"
                            >
                              ⭐
                            </button>
                          </div>
                        </div>
                        <div class="image-info">
                          <p class="image-name">{{ truncateFilename(image.name || 'imagen.jpg') }}</p>
                          <p class="image-size">{{ image.size ? formatFileSize(image.size) : 'Existente' }}</p>
                          <input 
                            v-model="image.descripcion"
                            placeholder="Descripción de la imagen"
                            class="image-description-input"
                            @click.stop
                          />
                        </div>
                        <button 
                          type="button" 
                          class="remove-image-btn"
                          @click.stop="removeUploadedImage(index)"
                          title="Eliminar imagen"
                        >
                          ×
                        </button>
                      </div>
                      
                      <!-- Espacio para añadir más imágenes -->
                      <div 
                        class="add-more-container"
                        @click="triggerFileInput"
                        @dragover.prevent="handleAddMoreDragOver"
                        @drop.prevent="handleAddMoreDrop"
                        :title="uploadedImages.length >= 20 ? 'Límite máximo alcanzado' : 'Añadir más imágenes'"
                        :class="{ 'disabled': uploadedImages.length >= 20 }"
                      >
                        <div class="add-more-icon">+</div>
                        <p>Añadir más</p>
                        <p class="add-more-hint">Arrastra aquí</p>
                      </div>
                    </div>
                  </div>
                  
                  <!-- Input de archivo oculto -->
                  <input
                    type="file"
                    ref="fileInput"
                    @change="handleFileSelect"
                    multiple
                    accept="image/jpeg,image/png,image/gif,image/webp"
                    style="display: none;"
                  />
                </div>
                
                <!-- Botones de acción para imágenes -->
                <div class="upload-actions">
                  <button 
                    type="button" 
                    @click="triggerFileInput"
                    class="upload-btn"
                    :disabled="uploadedImages.length >= 20"
                  >
                    📁 Seleccionar Archivos
                  </button>
                  <button 
                    type="button" 
                    @click="uploadAllImages"
                    :disabled="uploading || uploadedImages.filter(img => !img.uploaded && !img.dropboxUrl && img.file).length === 0"
                    class="upload-all-btn"
                  >
                    {{ uploading ? 'Subiendo...' : `Subir ${uploadedImages.filter(img => !img.uploaded && !img.dropboxUrl && img.file).length} nueva(s)` }}
                  </button>
                  <button 
                    type="button" 
                    @click="autoSortImages"
                    :disabled="uploadedImages.length < 2"
                    class="sort-btn"
                  >
                    🔄 Ordenar Alfabéticamente
                  </button>
                </div>
                
                <!-- Información de subida -->
                <div v-if="uploadStats" class="upload-stats">
                  <p>✅ {{ uploadStats.success }} subidas correctamente</p>
                  <p v-if="uploadStats.failed > 0">❌ {{ uploadStats.failed }} con error</p>
                  <p v-if="uploadStats.pending > 0">⏳ {{ uploadStats.pending }} pendientes</p>
                  <div class="current-order" v-if="uploadedImages.length > 0">
                    <p><strong>Orden actual (arrastra para cambiar):</strong></p>
                    <div class="order-list">
                      <span 
                        v-for="(img, index) in uploadedImages.slice(0, 5)" 
                        :key="img.id"
                        class="order-item"
                      >
                        {{ index + 1 }}. {{ truncateFilename(img.name || 'imagen', 15) }}
                      </span>
                      <span v-if="uploadedImages.length > 5" class="order-more">
                        ... y {{ uploadedImages.length - 5 }} más
                      </span>
                    </div>
                  </div>
                </div>
              </div>

              <div class="form-actions">
                <button type="submit" class="update-button" :disabled="uploading">
                  {{ uploading ? 'Subiendo imágenes...' : 'Actualizar Propiedad' }}
                </button>
                <button type="button" @click="cancelEdit" class="cancel-button">
                  Cancelar
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>

      <!-- Modal de confirmación -->
      <div v-if="showModal" class="modal-overlay">
        <div class="modal-content">
          <h2>Confirmar cambios</h2>
          <p>¿Estás seguro de que deseas actualizar esta propiedad en alquiler?</p>
          <div class="modal-buttons">
            <button @click="confirmEdit" class="confirm-button">Sí, actualizar</button>
            <button @click="cancelEdit" class="cancel-button">Cancelar</button>
          </div>
        </div>
      </div>

      <!-- Modal de éxito -->
      <div v-if="showSuccessModal" class="modal-overlay">
        <div class="modal-content success-modal">
          <h2>¡Actualización exitosa!</h2>
          <p>La propiedad en alquiler ha sido actualizada correctamente.</p>
          <button @click="redirectToAdmin" class="success-button">
            Volver al panel
          </button>
        </div>
      </div>

      <!-- Modal de carga de imágenes -->
      <div v-if="showLoadingModal" class="modal-overlay">
        <div class="modal-content">
          <h2>Subiendo Imágenes</h2>
          <div class="loading-spinner"></div>
          <p>Por favor espera mientras se suben las imágenes...</p>
          <p class="loading-progress">
            {{ currentUpload }} de {{ totalUploads }} imágenes
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import A_Sidebar from '../components/A_Sidebar.vue';
import { ref, onMounted, reactive } from 'vue';
import { useRouter } from 'vue-router';
import { generateClient } from 'aws-amplify/data';
import type { Schema } from '../../amplify/data/resource';
import { Dropbox } from 'dropbox';

// Configuración de Dropbox
const dbx = new Dropbox({
  clientId: 'uwu9drc8e3gpxwo',
  clientSecret: '4egf1e2bb93ddtc',
  refreshToken: 'KH6pPVv6uysAAAAAAAAAAROWpiU66a2szKfQxCg0U3vHOxlLNfsy4wOWDBRKPQuU',
});

const client = generateClient<Schema>();
const router = useRouter();

// Datos reactivos
const propiedades = ref<any[]>([]);
const loading = ref(true);
const selectedPropertyId = ref<string | null>(null);
const showModal = ref(false);
const showSuccessModal = ref(false);
const errorMessage = ref<string>('');
const showLoadingModal = ref(false);

// Variables para Drag and Drop
const fileInput = ref<HTMLInputElement | null>(null);
const isDragover = ref(false);
const uploading = ref(false);
const currentUpload = ref(0);
const totalUploads = ref(0);

// Variables para reordenamiento por drag & drop
const draggingImageId = ref<string | null>(null);
const dragOverImageId = ref<string | null>(null);
const isDragging = ref(false);

// Interfaz para imágenes
interface UploadedImage {
  id: string;
  file?: File;
  name: string;
  size?: number;
  previewUrl: string;
  descripcion: string;
  dropboxUrl: string;
  uploading: boolean;
  uploaded: boolean;
  error: boolean;
  progress: number;
  isExisting?: boolean;
}

// Datos de imágenes para edición
const uploadedImages = ref<UploadedImage[]>([]);
const uploadStats = ref<{ success: number; failed: number; pending: number } | null>(null);

// Formulario de edición
const formData = reactive({
  direccion: '',
  ubicacion: '',
  tipoPropiedad: '',
  precioAlquiler: 0,
  moneda: 'ARS',
  precioExpensas: 0,
  habitaciones: 0,
  banos: 0,
  cochera: 'No',
  metrosCuadradosTotales: 0,
  metrosCuadradosCubiertos: 0,
  mapLink: '',
  youtubeVideoUrl: '',
  descripcion: '',
});

const sinExpensas = ref(false);

// ================= FUNCIONES PARA GESTIÓN DE IMÁGENES =================
let isProcessingDrop = false;

const triggerFileInput = () => {
  if (uploadedImages.value.length >= 20) {
    alert('Has alcanzado el límite máximo de 20 imágenes');
    return;
  }
  fileInput.value?.click();
};

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  if (target.files && target.files.length > 0) {
    processFiles(Array.from(target.files));
  }
  target.value = '';
};

const handleDragOver = (event: DragEvent) => {
  event.preventDefault();
  isDragover.value = true;
};

const handleDragLeave = (event: DragEvent) => {
  event.preventDefault();
  const target = event.target as HTMLElement;
  const dropzone = target.closest('.dropzone');
  
  if (!dropzone?.contains(event.relatedTarget as Node)) {
    isDragover.value = false;
  }
};

const handleDrop = (event: DragEvent) => {
  event.preventDefault();
  event.stopPropagation();
  isDragover.value = false;
  
  if (isProcessingDrop) return;
  isProcessingDrop = true;
  
  if (uploadedImages.value.length >= 20) {
    alert('Has alcanzado el límite máximo de 20 imágenes');
    isProcessingDrop = false;
    return;
  }
  
  if (event.dataTransfer?.files) {
    const remainingSlots = 20 - uploadedImages.value.length;
    const files = Array.from(event.dataTransfer.files).slice(0, remainingSlots);
    
    if (files.length === 0) {
      alert(`Máximo 20 imágenes permitidas. Ya tienes ${uploadedImages.value.length}`);
      isProcessingDrop = false;
      return;
    }
    
    processFiles(files);
  }
  
  setTimeout(() => {
    isProcessingDrop = false;
  }, 100);
};

const handleDropzoneClick = (event: MouseEvent) => {
  const target = event.target as HTMLElement;
  if (!target.closest('.image-preview-container') && 
      !target.closest('.add-more-container') &&
      !target.closest('.reorder-instructions')) {
    triggerFileInput();
  }
};

const handleAddMoreDragOver = (event: DragEvent) => {
  event.preventDefault();
  if (uploadedImages.value.length < 20) {
    const container = event.currentTarget as HTMLElement;
    container.classList.add('drag-over');
  }
};

const handleAddMoreDrop = (event: DragEvent) => {
  event.preventDefault();
  event.stopPropagation();
  
  const container = event.currentTarget as HTMLElement;
  container.classList.remove('drag-over');
  
  handleDrop(event);
};

const handleGridDrop = (event: DragEvent) => {
  event.preventDefault();
  event.stopPropagation();
  isDragover.value = false;
};

const processFiles = (files: File[]) => {
  const imageFiles = files.filter(file => {
    if (!file.type.startsWith('image/')) {
      alert(`El archivo "${file.name}" no es una imagen válida`);
      return false;
    }
    
    const validTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
    if (!validTypes.includes(file.type)) {
      alert(`El formato de "${file.name}" no está soportado. Use JPG, PNG o GIF`);
      return false;
    }
    
    if (file.size > 10 * 1024 * 1024) {
      alert(`"${file.name}" supera el tamaño máximo de 10MB`);
      return false;
    }
    
    const existingImage = uploadedImages.value.find(img => 
      img.name === file.name && img.size === file.size
    );
    
    if (existingImage) {
      alert(`"${file.name}" ya ha sido agregado`);
      return false;
    }
    
    return true;
  });

  if (imageFiles.length === 0) return;

  let processedCount = 0;
  
  const processNextFile = () => {
    if (processedCount >= imageFiles.length) {
      updateUploadStats();
      return;
    }
    
    const file = imageFiles[processedCount];
    processedCount++;
    
    const reader = new FileReader();
    reader.onload = (e) => {
      const imageId = Date.now() + '-' + Math.random().toString(36).substr(2, 9) + '-' + processedCount;
      
      const existingImage = uploadedImages.value.find(img => 
        img.name === file.name && img.size === file.size
      );
      
      if (!existingImage) {
        uploadedImages.value.push({
          id: imageId,
          file,
          name: file.name,
          size: file.size,
          previewUrl: e.target?.result as string,
          descripcion: '',
          dropboxUrl: '',
          uploading: false,
          uploaded: false,
          error: false,
          progress: 0,
          isExisting: false
        });
      }
      
      processNextFile();
    };
    reader.onerror = () => {
      alert(`Error al leer el archivo "${file.name}"`);
      processNextFile();
    };
    reader.readAsDataURL(file);
  };
  
  processNextFile();
};

// ================= FUNCIONES PARA REORDENAR IMÁGENES =================
const handleImageDragStart = (event: DragEvent, image: UploadedImage) => {
  if (isDragging.value) return;
  
  isDragging.value = true;
  draggingImageId.value = image.id;
  event.dataTransfer?.setData('text/plain', image.id);
  event.dataTransfer!.effectAllowed = 'move';
  
  const element = event.target as HTMLElement;
  element.classList.add('dragging-active');
  
  event.stopPropagation();
};

const handleImageDragOver = (event: DragEvent, image: UploadedImage) => {
  event.preventDefault();
  
  if (draggingImageId.value !== image.id) {
    event.dataTransfer!.dropEffect = 'move';
  }
};

const handleImageDragEnter = (event: DragEvent, image: UploadedImage) => {
  event.preventDefault();
  
  if (!isDragging.value || draggingImageId.value === image.id) return;
  
  if (dragOverImageId.value !== image.id) {
    dragOverImageId.value = image.id;
  }
};

const handleImageDragLeave = (event: DragEvent) => {
  if (!isDragging.value) return;
  
  const target = event.currentTarget as HTMLElement;
  const relatedTarget = event.relatedTarget as HTMLElement;
  
  if (!target.contains(relatedTarget)) {
    dragOverImageId.value = null;
  }
};

const handleImageDrop = (event: DragEvent, targetImage: UploadedImage) => {
  event.preventDefault();
  event.stopPropagation();
  
  if (!isDragging.value) return;
  
  const draggedImageId = event.dataTransfer?.getData('text/plain');
  if (!draggedImageId || draggedImageId === targetImage.id) {
    resetDragState();
    return;
  }
  
  const draggedIndex = uploadedImages.value.findIndex(img => img.id === draggedImageId);
  const targetIndex = uploadedImages.value.findIndex(img => img.id === targetImage.id);
  
  if (draggedIndex !== -1 && targetIndex !== -1) {
    const newImages = [...uploadedImages.value];
    const [draggedItem] = newImages.splice(draggedIndex, 1);
    newImages.splice(targetIndex, 0, draggedItem);
    uploadedImages.value = newImages;
  }
  
  resetDragState();
};

const handleImageDragEnd = (event: DragEvent) => {
  event.preventDefault();
  resetDragState();
  
  const element = event.target as HTMLElement;
  element.classList.remove('dragging-active');
};

const resetDragState = () => {
  isDragging.value = false;
  draggingImageId.value = null;
  dragOverImageId.value = null;
  
  document.querySelectorAll('.dragging-active').forEach(el => {
    el.classList.remove('dragging-active');
  });
};

const moveImageUp = (index: number) => {
  if (index > 0) {
    const newImages = [...uploadedImages.value];
    const temp = newImages[index];
    newImages[index] = newImages[index - 1];
    newImages[index - 1] = temp;
    uploadedImages.value = newImages;
  }
};

const moveImageDown = (index: number) => {
  if (index < uploadedImages.value.length - 1) {
    const newImages = [...uploadedImages.value];
    const temp = newImages[index];
    newImages[index] = newImages[index + 1];
    newImages[index + 1] = temp;
    uploadedImages.value = newImages;
  }
};

const setAsMainImage = (index: number) => {
  if (index > 0) {
    const newImages = [...uploadedImages.value];
    const mainImage = newImages[index];
    newImages.splice(index, 1);
    newImages.unshift(mainImage);
    uploadedImages.value = newImages;
  }
};

const autoSortImages = () => {
  const newImages = [...uploadedImages.value];
  newImages.sort((a, b) => {
    return a.name.localeCompare(b.name);
  });
  uploadedImages.value = newImages;
};

// ================= FUNCIONES DROPBOX =================
const uploadImageToDropbox = async (image: UploadedImage): Promise<string> => {
  return new Promise(async (resolve, reject) => {
    try {
      if (!image.file) {
        // Si no hay archivo nuevo, usar la URL existente
        resolve(image.dropboxUrl);
        return;
      }

      const timestamp = new Date().getTime();
      const randomString = Math.random().toString(36).substring(2, 8);
      const fileExtension = image.name.split('.').pop();
      const propertyId = formData.direccion.substring(0, 10).replace(/[^a-zA-Z0-9]/g, '') || 'propiedad';
      
      const uniqueFileName = `propiedad_alquiler_${propertyId}_${timestamp}_${randomString}.${fileExtension}`;
      const dropboxPath = `/propiedades/alquiler/${propertyId}/${uniqueFileName}`;

      const response = await dbx.filesUpload({
        path: dropboxPath,
        contents: image.file,
      });

      const sharedLinkResponse = await dbx.sharingCreateSharedLinkWithSettings({
        path: response.result.path_lower || '',
      });

      const directLink = sharedLinkResponse.result.url
        .replace('www.dropbox.com', 'dl.dropboxusercontent.com')
        .replace('?dl=0', '?raw=1');

      resolve(directLink);
    } catch (error) {
      console.error('Error al subir imagen a Dropbox:', error);
      reject(error);
    }
  });
};

const uploadAllImages = async () => {
  const imagesToUpload = uploadedImages.value.filter(img => !img.uploaded && !img.dropboxUrl && img.file);

  if (imagesToUpload.length === 0) {
    updateUploadStats();
    return;
  }

  uploading.value = true;
  showLoadingModal.value = true;
  currentUpload.value = 0;
  totalUploads.value = imagesToUpload.length;

  for (let i = 0; i < imagesToUpload.length; i++) {
    const image = imagesToUpload[i];
    image.uploading = true;
    image.progress = 0;
    currentUpload.value = i + 1;

    const progressInterval = setInterval(() => {
      if (image.progress < 90) {
        image.progress += 10;
      }
    }, 200);

    try {
      const dropboxUrl = await uploadImageToDropbox(image);
      image.dropboxUrl = dropboxUrl;
      image.uploaded = true;
      image.error = false;
    } catch (error) {
      image.error = true;
      image.uploaded = false;
    } finally {
      image.uploading = false;
      image.progress = 100;
      clearInterval(progressInterval);
    }

    updateUploadStats();
  }

  uploading.value = false;
  showLoadingModal.value = false;
};

// ================= FUNCIONES AUXILIARES =================
const updateUploadStats = () => {
  const stats = {
    success: uploadedImages.value.filter(img => img.uploaded && !img.error).length,
    failed: uploadedImages.value.filter(img => img.error).length,
    pending: uploadedImages.value.filter(img => !img.uploaded && !img.error && img.file).length
  };
  uploadStats.value = stats;
};

const removeUploadedImage = (index: number) => {
  if (confirm('¿Estás seguro de eliminar esta imagen?')) {
    const newImages = [...uploadedImages.value];
    newImages.splice(index, 1);
    uploadedImages.value = newImages;
    updateUploadStats();
  }
};

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
};

const truncateFilename = (filename: string, maxLength: number = 20): string => {
  if (filename.length <= maxLength) return filename;
  const extension = filename.split('.').pop() || '';
  const nameWithoutExt = filename.substring(0, filename.length - extension.length - 1);
  const truncatedName = nameWithoutExt.substring(0, maxLength - extension.length - 3);
  return truncatedName + '...' + extension;
};

const safeJsonParse = (str: string) => {
  try {
    return JSON.parse(str);
  } catch (e) {
    console.error('Error al parsear imágenes:', e);
    return [];
  }
};

const handleExpensasChange = () => {
  if (sinExpensas.value) {
    formData.precioExpensas = 0;
  }
};

// ================= FUNCIONES PRINCIPALES =================
const loadProperties = async () => {
  try {
    loading.value = true;
    errorMessage.value = '';
    
    const { data: rawProperties, errors } = await client.models.PropiedadAlquiler.list();
    
    if (errors) {
      console.error('Errores al cargar propiedades:', errors);
      errorMessage.value = 'Error técnico al cargar propiedades';
      return;
    }

    // REMOVER FILTRO PROBLEMÁTICO - Mostrar todas las propiedades
    propiedades.value = (rawProperties || [])
      .filter(prop => prop && prop.id) // Solo filtrar propiedades nulas o sin ID
      .map(prop => ({
        ...prop,
        tipoPropiedad: prop.tipoPropiedad || 'No especificado',
        moneda: prop.moneda || 'ARS', // Asignar valor por defecto si no tiene
        precioAlquiler: prop.precioAlquiler || 0,
        ubicacion: prop.ubicacion || 'No especificada',
        imagenes: prop.imagenes ? safeJsonParse(prop.imagenes) : []
      }));

    console.log('Propiedades de alquiler cargadas:', propiedades.value.length);
    console.log('Primera propiedad:', propiedades.value[0]);

  } catch (error) {
    console.error('Error inesperado:', error);
    errorMessage.value = 'Error inesperado al cargar propiedades';
  } finally {
    loading.value = false;
  }
};

const loadPropertyData = async (id: string) => {
  try {
    const { data: propiedad, errors } = await client.models.PropiedadAlquiler.get({ id });
    
    if (errors || !propiedad) {
      console.error('Error al cargar propiedad:', errors);
      return;
    }
    
    // Resetear formulario
    Object.assign(formData, {
      direccion: '',
      ubicacion: '',
      tipoPropiedad: '',
      precioAlquiler: 0,
      moneda: 'ARS',
      precioExpensas: 0,
      habitaciones: 0,
      banos: 0,
      cochera: 'No',
      metrosCuadradosTotales: 0,
      metrosCuadradosCubiertos: 0,
      mapLink: '',
      youtubeVideoUrl: '',
      descripcion: '',
    });
    
    // Llenar con datos existentes
    if (propiedad) {
      Object.assign(formData, {
        direccion: propiedad.direccion || '',
        ubicacion: propiedad.ubicacion || '',
        tipoPropiedad: propiedad.tipoPropiedad || '',
        precioAlquiler: propiedad.precioAlquiler || 0,
        moneda: propiedad.moneda || 'ARS',
        precioExpensas: propiedad.precioExpensas || 0,
        habitaciones: propiedad.habitaciones || 0,
        banos: propiedad.banos || 0,
        cochera: propiedad.cochera || 'No',
        metrosCuadradosTotales: propiedad.metrosCuadradosTotales || 0,
        metrosCuadradosCubiertos: propiedad.metrosCuadradosCubiertos || 0,
        mapLink: propiedad.mapLink || '',
        youtubeVideoUrl: propiedad.youtubeVideoUrl || '',
        descripcion: propiedad.descripcion || '',
      });
      
      sinExpensas.value = !propiedad.precioExpensas || propiedad.precioExpensas <= 0;
    }
    
    // Cargar imágenes existentes
    if (propiedad?.imagenes) {
      const imagenesExistentes = safeJsonParse(propiedad.imagenes);
      uploadedImages.value = imagenesExistentes.map((img: any, index: number) => ({
        id: `existing-${index}-${Date.now()}`,
        name: img.descripcion || `imagen-${index + 1}`,
        previewUrl: img.url,
        descripcion: img.descripcion || '',
        dropboxUrl: img.url,
        uploading: false,
        uploaded: true,
        error: false,
        progress: 100,
        isExisting: true
      }));
    } else {
      uploadedImages.value = [];
    }
    
    updateUploadStats();
    
  } catch (error) {
    console.error('Error al cargar datos de la propiedad:', error);
  }
};

const selectProperty = async (id: string) => {
  selectedPropertyId.value = id;
  await loadPropertyData(id);
};

// ================= ACTUALIZAR PROPIEDAD =================
const updateProperty = async () => {
  try {
    if (!selectedPropertyId.value) return;

    // Verificar si hay imágenes nuevas pendientes de subir
    const pendingImages = uploadedImages.value.filter(img => !img.uploaded && !img.dropboxUrl && img.file);
    if (pendingImages.length > 0) {
      const uploadNow = confirm('Hay imágenes pendientes de subir. ¿Deseas subirlas ahora?');
      if (uploadNow) {
        await uploadAllImages();
      } else {
        const proceed = confirm('¿Continuar sin subir las imágenes pendientes? Las imágenes no subidas no se guardarán.');
        if (!proceed) return;
      }
    }

    // Preparar datos de imágenes para guardar
    const imagesToSave = uploadedImages.value
      .filter(img => img.dropboxUrl) // Solo imágenes con URL
      .map(img => ({
        url: img.dropboxUrl,
        descripcion: img.descripcion || img.name || 'Imagen de propiedad'
      }));

    // Actualizar la propiedad
    await client.models.PropiedadAlquiler.update({
      id: selectedPropertyId.value,
      direccion: formData.direccion,
      ubicacion: formData.ubicacion,
      tipoPropiedad: formData.tipoPropiedad,
      precioAlquiler: formData.precioAlquiler,
      moneda: formData.moneda,
      precioExpensas: formData.precioExpensas,
      habitaciones: formData.habitaciones,
      banos: formData.banos,
      cochera: formData.cochera,
      metrosCuadradosTotales: formData.metrosCuadradosTotales,
      metrosCuadradosCubiertos: formData.metrosCuadradosCubiertos,
      mapLink: formData.mapLink,
      youtubeVideoUrl: formData.youtubeVideoUrl,
      descripcion: formData.descripcion,
      imagenes: JSON.stringify(imagesToSave)
    });

    showModal.value = false;
    showSuccessModal.value = true;
    await loadProperties(); // Recargar la lista
  } catch (error) {
    console.error('Error al actualizar la propiedad:', error);
    alert('Error al actualizar la propiedad. Por favor, inténtalo de nuevo.');
  }
};

const confirmEdit = () => {
  updateProperty();
};

const cancelEdit = () => {
  showModal.value = false;
  selectedPropertyId.value = null;
  uploadedImages.value = [];
  uploadStats.value = null;
};

const redirectToAdmin = () => {
  showSuccessModal.value = false;
  router.push('/plataforma-administrador');
};

onMounted(() => {
  loadProperties();
});
</script>

<style scoped>
/* Estilos base consistentes con las otras vistas */
.container {
  display: flex;
  min-height: 100vh;
}

.content {
  margin-left: 220px;
  padding: 20px;
  width: calc(100% - 220px);
  position: relative;
}

.logo-plataforma {
  width: 500px;
  padding: 5px;
}

/* Estilos para el listado de propiedades */
.property-list-item {
  margin-bottom: 30px;
}

.property-item {
  background: white;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 10px;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.property-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

.property-details {
  display: flex;
  gap: 15px;
  margin-top: 8px;
  color: #555;
}

.property-details span {
  display: flex;
  align-items: center;
}

.property-status {
  margin-top: 10px;
}

.status-badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.status-badge.disponible {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.status-badge.reservado {
  background-color: #fff3e0;
  color: #f57c00;
}

.status-badge.alquilado {
  background-color: #ffebee;
  color: #c62828;
}

/* Estilos para el formulario de edición */
.property-edit-form {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 25px;
  margin-top: 15px;
}

.edit-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-section-title {
  font-size: 18px;
  font-weight: bold;
  margin: 20px 0 15px;
  padding-bottom: 8px;
  border-bottom: 2px solid #0014ac;
  color: #0014ac;
}

.form-row {
  display: flex;
  gap: 20px;
}

.form-row .form-group {
  flex: 1;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #333;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.form-group textarea {
  min-height: 100px;
  resize: vertical;
}

/* Estilos para el área de Drag and Drop (copiados del componente original) */
.dropzone {
  border: 2px dashed #ccc;
  border-radius: 8px;
  padding: 40px 20px;
  text-align: center;
  transition: all 0.3s ease;
  background-color: #fafafa;
  margin-bottom: 15px;
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  cursor: pointer;
}

.dropzone.dragover {
  border-color: #0014ac;
  background-color: rgba(0, 20, 172, 0.05);
  border-style: solid;
}

.dropzone.has-images {
  padding: 20px;
  min-height: auto;
  cursor: default;
}

.dropzone-empty {
  color: #666;
  cursor: pointer;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
}

.dropzone-empty:hover {
  color: #0014ac;
  transform: translateY(-2px);
}

.dropzone-icon {
  font-size: 48px;
  margin-bottom: 15px;
  opacity: 0.7;
  transition: all 0.3s;
}

.dropzone-empty:hover .dropzone-icon {
  opacity: 1;
  transform: scale(1.1);
}

.dropzone-subtext {
  color: #888;
  font-size: 14px;
  margin-top: 5px;
}

.dropzone-hint {
  font-size: 12px;
  color: #999;
  margin-top: 10px;
  max-width: 300px;
  margin-left: auto;
  margin-right: auto;
}

/* Instrucciones de reordenamiento */
.reorder-instructions {
  margin-bottom: 20px;
  padding: 15px;
  background: #e8f4ff;
  border-radius: 8px;
  border-left: 4px solid #0014ac;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.instructions-content {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.reorder-hint {
  color: #0014ac;
  font-size: 14px;
  font-weight: 500;
}

.image-count {
  color: #5d6d7e;
  font-size: 13px;
  font-weight: 500;
}

.add-more-button {
  padding: 8px 16px;
  background: #0014ac;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
  white-space: nowrap;
}

.add-more-button:hover:not(:disabled) {
  background: #001080;
  transform: translateY(-1px);
}

.add-more-button:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
}

/* Estilos para previsualización de imágenes */
.dropzone-preview {
  width: 100%;
}

.images-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
  position: relative;
}

.image-preview-container {
  position: relative;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
  background: white;
  transition: all 0.3s;
  cursor: grab;
  user-select: none;
  touch-action: none;
  display: flex;
  flex-direction: column;
  min-height: 280px;
}

.image-preview-container:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.image-preview-container:active {
  cursor: grabbing;
}

.image-preview-container.is-main {
  border: 2px solid #FFC107;
  box-shadow: 0 0 0 1px rgba(255, 193, 7, 0.3);
  background: linear-gradient(145deg, #fffde7, #ffffff);
}

.image-preview-container.dragging {
  opacity: 0.3;
  border: 2px dashed #0014ac;
  transform: rotate(3deg);
  box-shadow: 0 8px 25px rgba(0, 20, 172, 0.2);
}

.image-preview-container.drag-over {
  border: 2px solid #4CAF50;
  background-color: rgba(76, 175, 80, 0.05);
}

.image-preview-container.dragging-active {
  opacity: 0.7;
  transform: scale(0.98);
  box-shadow: 0 4px 15px rgba(0, 20, 172, 0.2);
}

/* Número de orden */
.image-order {
  position: absolute;
  top: -10px;
  left: -10px;
  width: 24px;
  height: 24px;
  background: linear-gradient(135deg, #0014ac, #4fc3f7);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: bold;
  z-index: 10;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

/* Indicador de imagen principal */
.main-indicator {
  position: absolute;
  top: -10px;
  right: -10px;
  background: linear-gradient(135deg, #FFC107, #FF9800);
  color: #333;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: bold;
  z-index: 10;
  white-space: nowrap;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.image-preview {
  position: relative;
  width: 100%;
  height: 150px;
  margin-bottom: 10px;
  border-radius: 6px;
  overflow: hidden;
  background-color: #f5f5f5;
  flex-shrink: 0;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}

.image-preview-container:hover .preview-image {
  transform: scale(1.05);
}

.upload-progress,
.upload-error,
.upload-success {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.75);
  color: white;
  padding: 8px;
  text-align: center;
  font-size: 12px;
  backdrop-filter: blur(2px);
}

.upload-progress {
  background: rgba(33, 150, 243, 0.85);
}

.upload-error {
  background: rgba(244, 67, 54, 0.85);
}

.upload-success {
  background: rgba(76, 175, 80, 0.85);
}

.progress-bar {
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 2px;
  overflow: hidden;
  margin-bottom: 5px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #8BC34A);
  transition: width 0.3s;
  border-radius: 2px;
}

.progress-text {
  font-size: 11px;
  font-weight: 500;
}

/* Indicador de arrastre */
.drag-indicator {
  position: absolute;
  top: 10px;
  left: 10px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 20px;
  cursor: move;
  z-index: 5;
  opacity: 0;
  transition: opacity 0.3s;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
  pointer-events: none;
}

.image-preview-container:hover .drag-indicator {
  opacity: 1;
}

/* Controles de orden en imagen */
.image-order-controls {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  gap: 5px;
  z-index: 5;
  opacity: 0;
  transition: opacity 0.3s;
}

.image-preview-container:hover .image-order-controls {
  opacity: 1;
}

.order-btn {
  width: 28px;
  height: 28px;
  border: none;
  border-radius: 4px;
  background: rgba(0, 0, 0, 0.75);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  transition: all 0.2s;
  backdrop-filter: blur(2px);
}

.order-btn:hover:not(:disabled) {
  transform: scale(1.1);
}

.order-btn.up-btn:hover:not(:disabled),
.order-btn.down-btn:hover:not(:disabled) {
  background: rgba(0, 20, 172, 0.9);
}

.order-btn.main-btn:hover:not(:disabled) {
  background: rgba(255, 193, 7, 0.9);
}

.order-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.image-info {
  text-align: left;
  flex: 1;
  display: flex;
  flex-direction: column;
  margin-top: 5px;
}

.image-name {
  font-size: 13px;
  font-weight: 500;
  margin: 0 0 5px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: #333;
}

.image-size {
  font-size: 11px;
  color: #666;
  margin: 0 0 10px 0;
}

.image-description-input {
  width: 100%;
  padding: 8px;
  font-size: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  margin-top: auto;
  pointer-events: auto;
  transition: all 0.3s;
}

.image-description-input:focus {
  outline: none;
  border-color: #0014ac;
  box-shadow: 0 0 0 2px rgba(0, 20, 172, 0.1);
}

.remove-image-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  width: 24px;
  height: 24px;
  border: none;
  border-radius: 50%;
  background: linear-gradient(135deg, #ff5252, #ff1744);
  color: white;
  cursor: pointer;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
  z-index: 15;
  opacity: 0;
  transform: scale(0.8);
}

.image-preview-container:hover .remove-image-btn {
  opacity: 1;
  transform: scale(1);
}

.remove-image-btn:hover {
  background: linear-gradient(135deg, #ff1744, #d50000);
  transform: scale(1.1) !important;
}

.add-more-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px dashed #ccc;
  border-radius: 8px;
  padding: 30px 20px;
  cursor: pointer;
  transition: all 0.3s;
  background: #f8f9fa;
  height: 100%;
  min-height: 280px;
  text-align: center;
}

.add-more-container:hover:not(.disabled) {
  border-color: #0014ac;
  background: rgba(0, 20, 172, 0.05);
  transform: translateY(-2px);
}

.add-more-container.drag-over:not(.disabled) {
  border-color: #4CAF50;
  background: rgba(76, 175, 80, 0.05);
  border-style: solid;
}

.add-more-container.disabled {
  cursor: not-allowed;
  opacity: 0.5;
  background: #f0f0f0;
}

.add-more-icon {
  font-size: 32px;
  color: #0014ac;
  margin-bottom: 15px;
  transition: all 0.3s;
}

.add-more-container:hover:not(.disabled) .add-more-icon {
  transform: scale(1.2);
}

.add-more-container p {
  margin: 0 0 8px 0;
  color: #0014ac;
  font-weight: 500;
  font-size: 15px;
}

.add-more-hint {
  font-size: 12px;
  color: #888;
  font-weight: normal;
  margin: 0;
}

/* Botones de acción */
.upload-actions {
  display: flex;
  gap: 10px;
  margin-top: 15px;
}

.upload-btn,
.upload-all-btn,
.sort-btn {
  flex: 1;
  padding: 12px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.upload-btn {
  background: #f0f0f0;
  color: #333;
  border: 1px solid #ddd;
}

.upload-btn:hover:not(:disabled) {
  background: #e0e0e0;
  transform: translateY(-1px);
}

.upload-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.upload-all-btn {
  background: #0014ac;
  color: white;
}

.upload-all-btn:hover:not(:disabled) {
  background: #001080;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 20, 172, 0.2);
}

.upload-all-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.sort-btn {
  background: #FF9800;
  color: white;
}

.sort-btn:hover:not(:disabled) {
  background: #F57C00;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(255, 152, 0, 0.2);
}

.sort-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* Estadísticas de subida */
.upload-stats {
  margin-top: 15px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 6px;
  border-left: 4px solid #0014ac;
}

.upload-stats p {
  margin: 5px 0;
  font-size: 13px;
  color: #555;
  display: flex;
  align-items: center;
  gap: 8px;
}

.current-order {
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px solid #eee;
}

.order-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 5px;
  align-items: center;
}

.order-item {
  font-size: 12px;
  color: #555;
  background: #f0f0f0;
  padding: 4px 10px;
  border-radius: 12px;
  border: 1px solid #ddd;
}

.order-more {
  font-size: 12px;
  color: #888;
  font-style: italic;
}

/* Estilos para los modales */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 10px;
  padding: 25px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
}

.success-modal {
  text-align: center;
}

.modal-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.confirm-button {
  background-color: #0014ac;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.cancel-button {
  background-color: #6c757d;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.success-button {
  background-color: #0014ac;
  color: white;
  margin-top: 20px;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

/* Botón de envío */
.update-button {
  background-color: #0014ac;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: background-color 0.3s;
}

.update-button:hover:not(:disabled) {
  background-color: #001080;
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 20, 172, 0.3);
}

.update-button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.form-actions {
  display: flex;
  gap: 15px;
  margin-top: 20px;
}

.cancel-button {
  background-color: #6c757d;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 600;
  transition: background-color 0.3s;
}

.cancel-button:hover {
  background-color: #5a6268;
  transform: translateY(-2px);
}

/* Mensajes */
.error-message {
  background-color: #ffebee;
  color: #c62828;
  padding: 12px;
  border-radius: 6px;
  margin-bottom: 20px;
  border-left: 4px solid #c62828;
}

.loading {
  text-align: center;
  padding: 30px;
  color: #555;
  font-size: 18px;
}

.no-properties {
  text-align: center;
  padding: 40px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.no-properties h3 {
  color: #555;
  margin-bottom: 10px;
}

.no-properties p {
  color: #777;
}

.checkbox-option {
  display: flex;
  align-items: center;
  margin-top: 8px;
}

.checkbox-option input[type="checkbox"] {
  width: auto;
  margin-right: 8px;
}

.checkbox-option label {
  font-weight: normal;
  margin-bottom: 0;
  color: #555;
  font-size: 14px;
}

/* Precio y moneda */
.price-container {
  display: flex;
  gap: 15px;
  align-items: center;
}

.price-input {
  flex: 1;
}

.currency-options {
  display: flex;
  gap: 10px;
}

.currency-option {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s;
  background: white;
}

.currency-option:hover {
  border-color: #0014ac;
  background: #f8f9ff;
}

.currency-option input[type="radio"] {
  margin: 0;
  accent-color: #0014ac;
}

.expensas-label {
  display: flex;
  align-items: center;
  gap: 8px;
}

.expensas-badge {
  background-color: #4CAF50;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: bold;
}

/* Spinner de carga */
.loading-spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #0014ac;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
  margin: 20px auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-progress {
  font-size: 14px;
  color: #666;
  margin-top: 10px;
}

@media screen and (max-width: 768px) {
  .form-row {
    flex-direction: column;
    gap: 0;
  }
  
  .content {
    margin-left: 0;
    width: 100%;
    padding: 15px;
  }
  
  .images-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 15px;
  }
  
  .upload-actions {
    flex-direction: column;
  }
  
  .price-container {
    flex-direction: column;
    align-items: stretch;
  }
  
  .currency-options {
    justify-content: center;
  }
  
  .image-order-controls {
    gap: 2px;
  }
  
  .order-btn {
    width: 24px;
    height: 24px;
    font-size: 12px;
  }
  
  .image-preview {
    height: 120px;
  }
  
  .drag-indicator {
    font-size: 16px;
  }
  
  .reorder-instructions {
    flex-direction: column;
    gap: 10px;
    text-align: center;
  }
  
  .add-more-button {
    width: 100%;
  }
  
  .form-actions {
    flex-direction: column;
  }
}
</style>