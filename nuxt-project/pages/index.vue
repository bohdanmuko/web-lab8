<template>
  <div class="home-container">
    <div class="content-wrapper">
      <h1 class="page-title">Головна сторінка</h1>
      <NuxtLink to="/products" class="products-link">
        Перейти до списку продуктів
        <span class="arrow">→</span>
      </NuxtLink>
    </div>
  </div>
</template>

<script setup>
useHead({
  title: 'Головна сторінка'
})
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');

.home-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7ff 0%, #e9eeff 100%);
  font-family: 'Poppins', sans-serif;
  padding: 20px;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  background-color: white;
  padding: 40px;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  max-width: 600px;
  width: 100%;
  position: relative;
  overflow: hidden;
}

.content-wrapper::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 6px;
  background: linear-gradient(90deg, #6e8efb, #a777e3);
}

.page-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #333;
  margin-bottom: 30px;
  position: relative;
  padding-bottom: 15px;
}

.page-title::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 4px;
  background: linear-gradient(90deg, #6e8efb, #a777e3);
  border-radius: 2px;
}

.products-link {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 12px 24px;
  background: linear-gradient(135deg, #6e8efb, #a777e3);
  color: white;
  text-decoration: none;
  border-radius: 8px;
  font-weight: 500;
  font-size: 1rem;
  transition: all 0.3s ease;
  box-shadow: 0 4px 10px rgba(110, 142, 251, 0.3);
  position: relative;
  overflow: hidden;
}

.products-link::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #a777e3, #6e8efb);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.products-link:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 15px rgba(110, 142, 251, 0.4);
}

.products-link:hover::before {
  opacity: 1;
}

.products-link:active {
  transform: translateY(0);
  box-shadow: 0 2px 5px rgba(110, 142, 251, 0.3);
}

.arrow {
  margin-left: 8px;
  font-size: 1.2rem;
  transition: transform 0.3s ease;
  position: relative;
  z-index: 1;
}

.products-link:hover .arrow {
  transform: translateX(4px);
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .page-title {
    font-size: 2rem;
  }

  .content-wrapper {
    padding: 30px 20px;
  }
}

@media (max-width: 480px) {
  .page-title {
    font-size: 1.75rem;
  }

  .products-link {
    padding: 10px 20px;
    font-size: 0.9rem;
  }
}
</style>