---
layout: page
title: Photography
---

I use a <a href="http://camera-wiki.org/wiki/Mamiya_M645" target="_blank">Mamiya M645</a>, which is a medium-format SLR film camera for my photography. Mamiya stopped making these in 1975. I use it with the Mamiya Sekor C 80mm f/2.8 and the Sekor C 75-150mm f/4.5 lenses. For films, I prefer using <a href="https://grainsandsuch.co/kodak-portra-400-35-120">Kodak Portra 400</a>, <a href="https://www.analog.cafe/r/kodak-ektar-100-film-review-59np">Kodak Ektar 100</a>, and <a href="https://www.shopmoment.com/reviews/ilford-hp5-400-film-review">Ilford HP5 400</a>. I don't like to edit my pictures in lightroom, photoshop, or any other crapware, to preserve the sanctity of film photography. I use a variety of film scanners from Noritsu, Nikon, and Epson (none of which I own).


## Photo Gallery

<style>
/* Grid Layout */
.image-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  grid-gap: 10px;
  margin: 20px 0;
}

.image-item {
  overflow: hidden;
  position: relative;
  height: 0;
  padding-bottom: 75%; /* 4:3 Aspect Ratio */
  background-color: #f5f5f5;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.image-item:hover {
  transform: scale(1.03);
}

.image-item img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: opacity 0.3s ease;
}

/* Modal Styles */
.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.9);
  overflow: hidden;
}

.modal-content {
  position: relative;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  max-width: 90%;
}

.modal-image {
  max-height: 90vh;
  max-width: 90%;
  object-fit: contain;
}

.close {
  position: absolute;
  top: 15px;
  right: 25px;
  color: #f1f1f1;
  font-size: 40px;
  font-weight: bold;
  transition: 0.3s;
  z-index: 1001;
}

.close:hover,
.close:focus {
  color: #bbb;
  text-decoration: none;
  cursor: pointer;
}

.prev,
.next {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  padding: 16px;
  color: white;
  font-weight: bold;
  font-size: 30px;
  transition: 0.6s ease;
  border-radius: 0 3px 3px 0;
  user-select: none;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 1001;
}

.next {
  right: 0;
  border-radius: 3px 0 0 3px;
}

.prev {
  left: 0;
}

.prev:hover,
.next:hover {
  background-color: rgba(0, 0, 0, 0.8);
}

/* Responsive Adjustments */
@media (max-width: 768px) {
  .image-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  }
  
  .prev,
  .next {
    font-size: 20px;
    padding: 10px;
  }
  
  .close {
    font-size: 30px;
  }
}
</style>

<div class="image-grid" id="imageGrid">
  <!-- Images will be loaded here via JavaScript -->
</div>

<!-- Modal for displaying full-size images -->
<div id="imageModal" class="modal">
  <span class="close" onclick="closeModal()">&times;</span>
  <a class="prev" onclick="changeImage(-1)">&#10094;</a>
  <div class="modal-content">
    <img class="modal-image" id="modalImage">
  </div>
  <a class="next" onclick="changeImage(1)">&#10095;</a>
</div>

<script>
// Sample image data - replace with your actual images
const images = [
  'https://raw.githubusercontent.com/kingroryg/lprimeroo.github.io/refs/heads/master/assets/000047990002.jpg',
  'https://via.placeholder.com/1200x800?text=Photo+2',
  'https://via.placeholder.com/1200x800?text=Photo+3',
];

let currentIndex = 0;
const imageGrid = document.getElementById('imageGrid');
const modal = document.getElementById('imageModal');
const modalImage = document.getElementById('modalImage');

// Create image grid
function createImageGrid() {
  images.forEach((imageUrl, index) => {
    const imageItem = document.createElement('div');
    imageItem.className = 'image-item';
    imageItem.dataset.index = index;
    
    const img = document.createElement('img');
    img.dataset.src = imageUrl; // Use data-src for lazy loading
    img.alt = 'Photo ' + (index + 1);
    
    imageItem.appendChild(img);
    imageItem.addEventListener('click', () => openModal(index));
    imageGrid.appendChild(imageItem);
  });
  
  // Initialize lazy loading
  lazyLoadImages();
}

// Lazy loading implementation
function lazyLoadImages() {
  const lazyImages = document.querySelectorAll('.image-item img[data-src]');
  
  if ('IntersectionObserver' in window) {
    const imageObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          // Create a low-res version for the thumbnail by adding size parameters
          const thumbnailUrl = img.dataset.src.includes('?') ? 
            img.dataset.src + '&size=300x200' : 
            img.dataset.src + '?size=300x200';
          img.src = thumbnailUrl;
          img.removeAttribute('data-src');
          imageObserver.unobserve(img);
        }
      });
    });
    
    lazyImages.forEach(img => imageObserver.observe(img));
  } else {
    // Fallback for browsers that don't support IntersectionObserver
    lazyImages.forEach(img => {
      const thumbnailUrl = img.dataset.src.includes('?') ? 
        img.dataset.src + '&size=300x200' : 
        img.dataset.src + '?size=300x200';
      img.src = thumbnailUrl;
      img.removeAttribute('data-src');
    });
  }
}

// Open modal with image
function openModal(index) {
  currentIndex = index;
  modal.style.display = 'block';
  updateModalImage();
}

// Close modal
function closeModal() {
  modal.style.display = 'none';
}

// Change image in modal
function changeImage(step) {
  currentIndex = (currentIndex + step + images.length) % images.length;
  updateModalImage();
}

// Update modal image
function updateModalImage() {
  // Load the full-size image
  modalImage.src = images[currentIndex];
}

// Close modal when clicking outside the image
window.addEventListener('click', (event) => {
  if (event.target === modal) {
    closeModal();
  }
});

// Handle keyboard navigation
window.addEventListener('keydown', (event) => {
  if (modal.style.display === 'block') {
    if (event.key === 'ArrowLeft') {
      changeImage(-1);
    } else if (event.key === 'ArrowRight') {
      changeImage(1);
    } else if (event.key === 'Escape') {
      closeModal();
    }
  }
});

// Initialize the image grid when the page loads
document.addEventListener('DOMContentLoaded', createImageGrid);

// Initialize lazy loading on scroll
window.addEventListener('scroll', lazyLoadImages);

// Initialize on page load regardless of DOMContentLoaded state
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', createImageGrid);
} else {
  createImageGrid();
}
</script>
