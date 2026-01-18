# Testing Guide - Code Review Fixes

Complete testing procedures for all fixed issues.

---

## Test Environment Setup

### Requirements
- Modern web browser (Chrome, Firefox, Safari, or Edge)
- Local web server (use `Live Server` VS Code extension or Python: `python -m http.server 8000`)
- Developer Tools (F12)

### Browser DevTools Checks
- Console: No errors or warnings
- Network: All resources load (check for 404s)
- Application: localStorage data persists correctly

---

## 🔴 CRITICAL FIXES TESTING

### Test 1: Script Path Fix (cart.html)
**Issue Fixed:** Broken script path

**Steps:**
1. Navigate to `/cart.html`
2. Open DevTools Console (F12)
3. Check for any script loading errors

**Expected Result:**
- ✅ No "Failed to load resource" errors
- ✅ JavaScript functions are available in console
- ✅ console shows no errors

**Testing:**
```javascript
// In DevTools Console, verify script loaded:
typeof getCartItems === 'function'  // Should return: true
typeof addToCart === 'function'     // Should return: true
```

---

### Test 2: Brand Name Consistency
**Issue Fixed:** Inconsistent brand names

**Steps:**
1. Check page titles:
   - index.html title should contain "SoleStride"
   - cart.html title should be "Cart - SoleStride" (not "Outfit Store")
   - contact.html title should contain "SoleStride"

2. Check navigation logos:
   - All pages should show "👟 SoleStride" logo
   - No "OutfitStore" should appear anywhere

**Expected Result:**
- ✅ All pages consistently use "SoleStride"
- ✅ No brand name mismatches
- ✅ Logo appears correctly on all pages

**Testing:**
```javascript
document.title  // Check browser tab title
document.querySelector('.logo').textContent  // Check logo text
```

---

### Test 3: Hard-coded Cart Counts
**Issue Fixed:** Dynamic cart count updates

**Steps:**
1. Open DevTools Console
2. Execute:
```javascript
// Add item to cart
localStorage.setItem('cartItems', JSON.stringify([
    { id: '1', name: 'Test Shoe', price: 99.99, quantity: 2, image: 'test.jpg' }
]));
window.location.reload();
```

3. After page reloads, check cart count

**Expected Result:**
- ✅ Cart count shows "2" (not hard-coded "3" or "0")
- ✅ Count updates when you add items
- ✅ Count syncs across all pages (index, products, contact, cart)

**Testing:**
```javascript
// In DevTools Console:
document.querySelector('.cart-count').textContent  // Should show actual count
```

---

### Test 4: Filter UI in products.html
**Issue Fixed:** Missing filter controls

**Steps:**
1. Navigate to `/products.html`
2. Verify the following elements exist:
   - Category dropdown (`#category`)
   - Price range slider (`#price`)
   - Price value display (`#priceValue`)
   - Sort dropdown (`#sort`)

3. Test functionality:
   - Change category and verify products filter
   - Drag price slider and verify products update
   - Change sort and verify products reorder

**Expected Result:**
- ✅ All filter controls visible
- ✅ Category filter works
- ✅ Price range filter works
- ✅ Sort dropdown works
- ✅ Multiple filters work together
- ✅ Products grid updates correctly

**Testing:**
```javascript
// In DevTools Console on products.html:
document.getElementById('category')  // Should exist
document.getElementById('price')     // Should exist
document.getElementById('priceValue') // Should exist
document.getElementById('sort')       // Should exist
```

---

## 🟠 HIGH SEVERITY FIXES TESTING

### Test 5: XSS Vulnerability Protection
**Issue Fixed:** Safe HTML rendering in cart

**Steps:**
1. Add a malicious product to cart via console:
```javascript
const maliciousProduct = {
    id: '999',
    name: '<img src=x onerror="alert(\'XSS Attack!\')">',
    price: 99.99,
    quantity: 1,
    image: 'test.jpg'
};
addToCart(maliciousProduct);
```

2. Navigate to cart page
3. Check if alert appears

**Expected Result:**
- ✅ No alert appears (XSS blocked)
- ✅ Malicious code displayed as plain text
- ✅ Console shows no security warnings
- ✅ Cart renders safely

**Testing:**
```javascript
// The product name should be safe:
const cartItem = document.querySelector('.cart-item-name');
cartItem.textContent  // Should show literal text, not execute script
```

---

### Test 6: Error Handling in JSON Parse
**Issue Fixed:** Better error handling

**Steps:**
1. Corrupt localStorage:
```javascript
localStorage.setItem('cartItems', 'invalid{json}');
```

2. Reload page
3. Check console for error message

**Expected Result:**
- ✅ Console shows: "Corrupted cartItems in localStorage. Resetting."
- ✅ Page doesn't crash
- ✅ Cart initializes with empty state
- ✅ Corrupted data is cleared

---

### Test 7: Input Validation
**Issue Fixed:** Product data validation

**Steps:**
1. Try to add invalid product:
```javascript
addToCart({});  // Empty object
addToCart({ id: '1' });  // Missing name
addToCart({ id: '1', name: 'Test', price: -50 });  // Negative price
addToCart({ id: '1', name: 'Test', price: 'invalid' });  // Wrong type
```

2. Check notifications

**Expected Result:**
- ✅ Notification shows: "Invalid product information"
- ✅ Product is NOT added to cart
- ✅ Console shows error details
- ✅ Page continues to function

---

### Test 8: Price Calculation Error Handling
**Issue Fixed:** Safe price calculations

**Steps:**
1. Add item with invalid data to localStorage:
```javascript
const cart = [
    { id: '1', name: 'Good', price: 50, quantity: 2 },
    { id: '2', name: 'Bad', price: 'invalid', quantity: 1 }
];
localStorage.setItem('cartItems', JSON.stringify(cart));
```

2. Reload cart page
3. Check if total calculates safely

**Expected Result:**
- ✅ Price breakdown still displays
- ✅ Invalid item is skipped
- ✅ Good item is calculated: (50 * 2) = 100
- ✅ No calculation errors in console

---

### Test 9: Performance - DOM Updates
**Issue Fixed:** Optimized filter performance

**Steps:**
1. Open DevTools Performance tab
2. On products.html, trigger filter:
   - Change category
   - Adjust price slider
   - Change sort

3. Check DevTools > Performance > Record

**Expected Result:**
- ✅ Single reflow per action (not multiple)
- ✅ Smooth 60fps performance
- ✅ No janky animation
- ✅ Fast response to filter changes

---

## 🟡 MEDIUM SEVERITY FIXES TESTING

### Test 10: Constants and Magic Numbers
**Issue Fixed:** Named constants used

**Steps:**
1. In DevTools Console, check code:
```javascript
// Verify constants are defined:
NOTIFICATION_TIMEOUT  // Should be 3000
DEBOUNCE_WAIT         // Should be 300
SCROLL_TOP_THRESHOLD  // Should be 100
```

2. Observe notification timeout (should be 3 seconds)
3. Observe scroll-to-top appears after 100px scroll

**Expected Result:**
- ✅ All constants properly defined
- ✅ Notification shows for correct duration
- ✅ Scroll button appears at right threshold

---

### Test 11: Filter Integration
**Issue Fixed:** Proper filter functionality

**Steps:**
1. On products.html:
   - Select "Running" category
   - Set max price to $100
   - Sort by "Price: Low to High"

2. Verify results

**Expected Result:**
- ✅ Only "Running" shoes shown
- ✅ All shown shoes ≤ $100
- ✅ Products sorted by price ascending
- ✅ All three filters work together

---

### Test 12: Code Organization
**Issue Fixed:** Removed inline scripts

**Steps:**
1. Check products.html source code
2. Verify no `<script>` tag at end of file
3. Verify all functionality still works

**Expected Result:**
- ✅ No inline script in HTML
- ✅ All functionality works via main script.js
- ✅ Code is DRY (no duplication)

---

## 🟢 LOW SEVERITY CHECKS

### Test 13: Accessibility
**Issue Fixed:** Documented ARIA improvements

**Steps:**
1. Test with screen reader (NVDA, JAWS, or built-in)
2. Test keyboard navigation:
   - Tab through all elements
   - Use Enter to activate buttons
   - Use Arrow keys for sliders

**Expected Result:**
- ✅ All elements are keyboard accessible
- ✅ Focus indicators visible
- ✅ Screen reader announces elements correctly
- ✅ No keyboard traps

**Testing with DevTools:**
```javascript
// Check ARIA attributes:
document.querySelectorAll('[role]').length  // Show elements with roles
document.querySelectorAll('[aria-label]').length  // Show labeled elements
```

---

## 📱 RESPONSIVE TESTING

### Mobile (320px - 480px)
1. Open products.html on mobile or use DevTools device emulation
2. Test all filters work
3. Check layout is readable
4. Verify cart count visible

**Expected Result:**
- ✅ All elements responsive
- ✅ Filters stack properly
- ✅ Touch interactions work

### Tablet (481px - 768px)
1. Test on iPad size (768px width)
2. Verify product grid adjusts
3. Check navigation is accessible

**Expected Result:**
- ✅ Grid shows 2 columns
- ✅ Hamburger menu functional
- ✅ Touch friendly

### Desktop (1025px+)
1. Test on full desktop width
2. Verify all features working
3. Check hover effects

**Expected Result:**
- ✅ Grid shows 3+ columns
- ✅ Navigation fully visible
- ✅ All hover effects work

---

## 🔍 FULL WORKFLOW TESTING

### Complete User Journey Test

**Scenario: Customer buys shoes**

1. ✅ Land on index.html
   - Cart count shows "0"
   
2. ✅ Navigate to products.html
   - See "0" in cart count
   - Filters are present and functional
   
3. ✅ Filter and search
   - Select "Running" category
   - Set max price $150
   - Sort by price
   - Products update correctly
   
4. ✅ Add items to cart
   - Click "Add to Cart"
   - Notification shows
   - Cart count increases
   
5. ✅ View cart
   - Go to cart.html
   - Cart count syncs
   - Items display correctly
   - Price breakdown calculates
   - No errors in console
   
6. ✅ Apply coupon
   - Enter "SAVE10"
   - Discount applies
   - Total updates
   
7. ✅ Estimate shipping
   - Enter ZIP code
   - Shipping cost updates
   
8. ✅ Modify cart
   - Change quantity
   - Remove item
   - Totals recalculate
   - Cart count updates
   
9. ✅ Navigate back
   - Cart count persists
   - All pages consistent

---

## ✅ SIGN-OFF CHECKLIST

Before considering fixes complete, verify:

- [ ] No console errors on any page
- [ ] All 4 critical fixes working
- [ ] All 6 high severity fixes working
- [ ] All 5 medium severity checks passed
- [ ] All 3 low severity items addressed
- [ ] Responsive design working (mobile, tablet, desktop)
- [ ] Cart functionality complete
- [ ] Filter functionality complete
- [ ] Security validated
- [ ] Performance acceptable
- [ ] Accessibility tested
- [ ] Code is maintainable
- [ ] No duplicate code
- [ ] Error messages helpful

---

## 🐛 TROUBLESHOOTING

### Issue: Cart count shows 0 even after adding items
**Solution:**
```javascript
// Check localStorage:
localStorage.getItem('cartItems')  // Should show JSON array
// If empty, restart browser cache
```

### Issue: Filters not working
**Solution:**
```javascript
// Verify elements exist:
document.getElementById('category') !== null  // true
// Check console for JavaScript errors
```

### Issue: Cart page shows script error
**Solution:**
```javascript
// Verify script path:
// cart.html should load: <script src="./script.js" defer></script>
// NOT: <script src="../script.js" defer></script>
```

### Issue: XSS test shows alert
**Solution:**
- This means XSS protection is not working properly
- Review renderCartItems() function
- Ensure all innerHTML replaced with textContent

---

## Performance Benchmarks

**Target Metrics:**
- Page load time: < 2 seconds
- Filter response: < 200ms
- Cart operations: < 100ms
- No memory leaks: Monitor heap size

**Tools:**
- Chrome DevTools > Performance
- Lighthouse audit
- Web Vitals measurement

---

## Final Verification

Run this complete test in DevTools Console to verify all fixes:

```javascript
// Test 1: Script loaded
console.log('✅ Script loaded:', typeof getCartItems === 'function');

// Test 2: Constants defined  
console.log('✅ Constants defined:', NOTIFICATION_TIMEOUT === 3000);

// Test 3: Add product validation
const testProduct = { id: 'test', name: 'Test', price: 50 };
addToCart(testProduct);
console.log('✅ Product added to cart:', localStorage.getItem('cartItems') !== null);

// Test 4: Cart renders safely
const safeName = '<img src=x onerror="alert(123)">';
const unsafeProduct = { id: 'xss', name: safeName, price: 99, quantity: 1, image: 'x.jpg' };
localStorage.setItem('cartItems', JSON.stringify([unsafeProduct]));
console.log('✅ Safe rendering (check cart.html for literal text, not alert)');

// Test 5: Verify responsive breakpoints
console.log('✅ Breakpoints defined:', getComputedStyle(document.documentElement).getPropertyValue('--breakpoint-md'));
```

---

**All tests passed?** ✅ **Your code is ready!**

For questions, refer to CODE_REVIEW.md or FIXES_APPLIED.md.
