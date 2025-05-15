# Analytics & Usage Tracking

This document outlines how we instrument and measure key adoption and usage metrics for Shop Kart, so we can understand user behavior, identify friction points, and drive continuous improvement.

---

## 1. Goals & Objectives

- **User Engagement**  
  Track how users browse, search, and interact with products and the cart.

- **Conversion Funnel**  
  Measure drop-off at each step: product view → add to cart → checkout initiated → order placed.

- **Feature Adoption**  
  Understand usage of reviews, search suggestions, filters, and payment simulation.

- **Performance & Reliability**  
  Monitor API latencies, error rates, and frontend load times.

---

## 2. Tools & Platforms

| Tool / Service                 | Purpose                                                   | Why?                                          |
|--------------------------------|-----------------------------------------------------------|-----------------------------------------------|
| **Google Analytics 4**         | Pageviews, user sessions, traffic sources, funnels        | Industry standard; easy SPA integration      |
| **Mixpanel (optional)**        | Event-level tracking, user cohorts, advanced funnels      | Powerful analysis of feature usage           |
| **Azure Application Insights** | Backend request performance, error rates, dependency map  | Built-in with Azure; end-to-end tracing       |
| **Segment (optional)**         | Unified event routing to GA4, Mixpanel, custom sinks      | Single SDK → multiple analytics back-ends     |

---

## 3. Key Events & Properties

### 3.1 Page / Screen Views (GA4)

```js
gtag('event', 'page_view', {
  page_path: window.location.pathname,
  page_title: document.title
});
```

### 3.2 Custom Events
| Event Name           | Trigger                                                | Properties                               |
| -------------------- | ------------------------------------------------------ | ---------------------------------------- |
| `product_view`       | User navigates to `/products/:id`                      | `product_id`, `product_name`, `category` |
| `search`             | User submits the search form                           | `search_term`                            |
| `suggestion_click`   | User selects a suggestion in the search dropdown       | `suggestion_id`, `suggestion_name`       |
| `add_to_cart`        | Click “Add to Cart” on product list or details page    | `product_id`, `quantity`, `price`        |
| `remove_from_cart`   | Click “Remove” in mini-cart or cart page               | `item_id`, `product_id`, `quantity`      |
| `cart_view`          | Mini-cart dropdown opened or `/cart` page loaded       | `item_count`, `subtotal`                 |
| `checkout_initiated` | Click “Proceed to Checkout”                            | `item_count`, `subtotal`                 |
| `order_placed`       | Successful `POST /api/orders/checkout` response        | `order_id`, `order_total`, `item_count`  |
| `payment_simulated`  | Click “Pay Now” stub button on order confirmation page | `order_id`, `payment_status`             |
| `review_submitted`   | Successful `POST /api/products/:id/reviews`            | `product_id`, `rating`                   |
| `profile_updated`    | Successful profile update (name/email/password)        | `changed_fields`                         |

## 4. Funnels & Dashboards


1. Conversion Funnel
<ul>
  <li>Steps: product_view → add_to_cart → checkout_initiated → order_placed</li>

  <li>Set up in GA4 (or Mixpanel) to visualize drop-off percentages.</li>
  </ul>

2. Cart Abandonment
<ul>
   <li>Compare users with add_to_cart but no checkout_initiated within 24 hrs.</li>
     </ul>

4. Feature Adoption
 <ul>
   <li>% of users using search suggestions (suggestion_click) vs. plain searches.</li>
   
   <li>% of orders using “Pay Now” stub (payment_simulated).</li>
     </ul>

6. Error Monitoring
<ul>

   <li>Azure AI: track 5xx error rates on API endpoints, alert on >1% error rate.</li>
        </ul>

8. Performance Metrics
   <ul>

   <li>Frontend: Time to Interactive (TTI), Largest Contentful Paint (LCP) via Real User Monitoring (RUM).</li>
   
   <li>Backend: average & p95 response times for key routes (/api/cart, /api/orders/checkout) in App Insights.</li>
    </ul>
   
## 5. Implementation Steps

1. Install SDKs
   <ul><li>Frontend</li></ul>
    ```bash
   npm install react-ga4 @azure/application-insights-web
   ```

2. Install SDKs
   <ul><li>Backend</li></ul>
    ```bash
   pip install opencensus-ext-azure
   ```

2. Initialize GA4
```js
// src/utils/analytics.js
import ReactGA from 'react-ga4';
ReactGA.initialize('G-XXXXXXXXXX');
export default ReactGA;
```
3. Wrap SPA
```jsx
// src/App.jsx
import { useEffect } from 'react';
import ReactGA from './utils/analytics';
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();
  useEffect(() => {
    ReactGA.send({ hitType: 'pageview', page: location.pathname });
  }, [location]);
  return (…);
}

```
4. Track Custom Events

```js
   import ReactGA from '../utils/analytics';
// e.g. after add-to-cart succeeds:
ReactGA.event({
  category: 'Cart',
  action: 'Add to Cart',
  label: product.name,
  value: quantity
});
```
5. Configure Azure App Insights
```js
// src/index.js (frontend)
import { ApplicationInsights } from '@microsoft/applicationinsights-web';

const appInsights = new ApplicationInsights({
  config: { instrumentationKey: process.env.REACT_APP_AI_KEY }
});
appInsights.loadAppInsights();
appInsights.trackPageView();
```
```python
# app/__init__.py (backend)
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.trace import config_integration
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['requests', 'sqlalchemy'])
tracer = Tracer(
  exporter=AzureExporter(connection_string=os.getenv('APPINSIGHTS_CONN')),
  sampler=ProbabilitySampler(1.0)
)
```
6. Build Dashboards & Alerts
- In GA4: define funnels & custom reports.

- In Azure Portal: create Application Insights dashboards & set up alerts on error rates and response times.

## 6. Next Steps
- A/B testing on checkout UX.

- Cohort analysis of repeat buyers.

- Automated email triggers based on abandonment events.

With this instrumentation in place, Shop Kart will have full visibility into user behavior, performance, and reliability—enabling data-driven product decisions and continual optimization.



