# TEJA-SNEAKERS-
Teja sneakers 
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>TEJA SNEAKERS — Browse Sneakers</title>
  <link rel="stylesheet" href="styles.css" />
</head>
<body>
  <header class="site-header">
    <div class="container header-row">
      <div style="display:flex;align-items:center;gap:1rem;">
        <h1 class="logo">TEJA SNEAKERS</h1>
        <nav class="nav">
          <a href="index.html">Shop</a>
          <a href="admin.html">Manage (Admin)</a>
        </nav>
      </div>

      <div style="display:flex;align-items:center;gap:.75rem;">
        <!-- Visible phone for paying orders -->
        <div class="contact-pill">
          Call / Pay: <strong>+254708237022</strong>
        </div>

        <!-- WhatsApp link (opens WhatsApp chat) -->
        <a class="whatsapp-btn" href="https://wa.me/254708237022" target="_blank" rel="noopener">WhatsApp: +254708237022</a>
      </div>
    </div>
  </header>

  <main class="container">
    <section class="hero">
      <h2>Latest Sneakers</h2>
      <p>Browse our selection and place an order from any shoe card. For quick contact use WhatsApp: <strong>+254708237022</strong></p>
    </section>

    <section id="productGrid" class="product-grid" aria-live="polite">
      <!-- Products are rendered here by app.js -->
    </section>
  </main>

  <!-- Order Modal -->
  <div id="orderModal" class="modal" aria-hidden="true">
    <div class="modal-content">
      <button class="modal-close" id="closeOrderModal" title="Close">×</button>
      <h3 id="orderModalTitle">Order Shoe</h3>
      <div id="orderShoeInfo" class="order-shoe-info"></div>

      <form id="orderForm">
        <input type="hidden" name="productId" />
        <label>
          Your name
          <input type="text" name="customerName" required />
        </label>
        <label>
          Email
          <input type="email" name="customerEmail" required />
        </label>
        <label>
          Phone (we will notify this number). Examples: 0708..., 708..., +254708...
          <input type="tel" name="customerPhone" placeholder="+254708237022" required />
        </label>
        <label>
          Address
          <textarea name="customerAddress" rows="2" required></textarea>
        </label>
        <label>
          Quantity
          <input type="number" name="quantity" value="1" min="1" required />
        </label>

        <!-- Visible payment instructions -->
        <div class="payment-instructions card">
          <strong>Payment / Mobile Money number:</strong>
          <div class="muted" style="font-size:1.15rem;margin-top:0.25rem"><strong>+254708237022</strong></div>
          <div style="margin-top:0.5rem">After placing your order you can pay to the number above. We'll contact you via the phone number you provide (we will normalize local formats to international format where possible).</div>
        </div>

        <div class="form-actions">
          <button type="submit" class="btn-primary">Place Order</button>
          <button type="button" id="cancelOrder" class="btn-ghost">Cancel</button>
        </div>
        <p id="orderStatus" class="muted"></p>
      </form>
    </div>
  </div>

  <footer class="site-footer">
    <div class="container">
      <div style="display:flex;justify-content:space-between;align-items:center;gap:1rem;flex-wrap:wrap;">
        <small>© 2025 TEJA SNEAKERS — Demo site. Orders are stored locally (LocalStorage) unless server notifications are configured.</small>
        <div style="display:flex;gap:0.75rem;align-items:center;">
          <div class="contact-pill">Call / Pay: <strong>+254708237022</strong></div>
          <a class="whatsapp-btn" href="https://wa.me/254708237022" target="_blank" rel="noopener">Message on WhatsApp</a>
        </div>
      </div>
    </div>
  </footer>

  <script src="app.js"></script>
  <script>
    // Initialize page-specific behavior
    document.addEventListener('DOMContentLoaded', () => {
      if (typeof store !== 'undefined') {
        store.renderProductGrid('productGrid');
        // Order modal wiring
        const orderModal = document.getElementById('orderModal');
        const closeOrderModal = document.getElementById('closeOrderModal');
        const cancelOrder = document.getElementById('cancelOrder');
        function hideModal() { orderModal.setAttribute('aria-hidden', 'true'); }
        function showModal() { orderModal.setAttribute('aria-hidden', 'false'); }
        closeOrderModal.addEventListener('click', hideModal);
        cancelOrder.addEventListener('click', hideModal);

        // Delegated click handler for Order buttons
        document.getElementById('productGrid').addEventListener('click', (e) => {
          const btn = e.target.closest('button[data-action="order"]');
          if (!btn) return;
          const id = btn.getAttribute('data-id');
          const product = store.getProductById(id);
          if (!product) return alert('Product not found.');
          // populate modal
          document.querySelector('#orderForm input[name="productId"]').value = product.id;
          document.getElementById('orderModalTitle').textContent = 'Order: ' + product.name;
          document.getElementById('orderShoeInfo').innerHTML = `
            <div class="order-card">
              <img src="${product.image}" alt="${product.name}" />
              <div>
                <strong>${product.name}</strong>
                <p class="muted">${product.description || ''}</p>
                <p class="price">${store.formatPrice(product.price)}</p>
                <div style="margin-top:.5rem"><strong>Pay/Contact:</strong> +254708237022</div>
                <div style="margin-top:.25rem"><a class="whatsapp-inline" href="https://wa.me/254708237022" target="_blank" rel="noopener">Chat on WhatsApp</a></div>
              </div>
            </div>`;
          showModal();
        });

        // Order form submission
        document.getElementById('orderForm').addEventListener('submit', async (e) => {
          e.preventDefault();
          const form = e.target;
          const data = {
            productId: form.productId.value,
            customerName: form.customerName.value.trim(),
            customerEmail: form.customerEmail.value.trim(),
            customerPhone: form.customerPhone.value.trim(),
            customerAddress: form.customerAddress.value.trim(),
            quantity: Number(form.quantity.value),
          };
          document.getElementById('orderStatus').textContent = 'Placing order...';
          try {
            const result = store.placeOrder(data);
            // result contains { order, sendPromise } in the updated app.js
            document.getElementById('orderStatus').textContent = 'Order placed — thank you! Please pay to +254708237022. (Stored locally)';
            form.reset();
            // Attempt to notify server (if you set one up)
            if (result && result.sendPromise) {
              result.sendPromise.then(res => {
                console.log('Server notify result:', res);
              });
            }
            setTimeout(() => {
              document.getElementById('orderStatus').textContent = '';
              orderModal.setAttribute('aria-hidden', 'true');
            }, 1600);
          } catch (err) {
            document.getElementById('orderStatus').textContent = 'Failed to place order: ' + err.message;
          }
        });
      }
    });
  </script>
</body>
</html>
