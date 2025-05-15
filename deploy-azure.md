# 🚀 Deploying ShopKart to Azure

This guide walks you through provisioning all required Azure resources, deploying both frontend and backend, and gluing it together with networking, storage, and monitoring.

---

## 🔧 Service Catalog

| Azure Service                         | Purpose                                       | Advantages / Why We Chose It                                      |
|---------------------------------------|-----------------------------------------------|-------------------------------------------------------------------|
| **Resource Group**                    | Logical container for all ShopKart resources  | One-stop management, easy RBAC, lifecycle + cost tracking         |
| **Azure Database for PostgreSQL**     | Managed relational database                   | Built-in HA, automated backups, predictable performance           |
| **Azure Cache for Redis**             | In-memory cache for session/cart data         | Ultra-low latency, reduces DB load, easy scale-out                |
| **Storage Account (Static Website)**  | Host React SPA artifacts                      | Very low cost, global redundancy, native static website support   |
| **App Service Plan**                  | Compute tier for hosting Flask API            | Fully managed Linux hosting, auto patching, scale up/down         |
| **Web App (Flask REST API)**          | Run our Python backend                        | Zero-downtime deploys, CI/CD integrations, custom domains         |
| **Azure CDN (Optional)**              | CDN fronting for static assets                | Global PoPs, HTTP/2, caching rules, offloads origin traffic       |
| **Application Insights**              | Application monitoring & telemetry            | Live metrics, distributed traces, built-in alerts & dashboards    |

---

## ✅ Prerequisites
- **Local dev** uses SQLite out of the box (no setup needed – data lives in `instance/shopkart.db`).
- **For Azure prod**, we’ll provision Azure Database for PostgreSQL and set `DATABASE_URL` accordingly.
- **Azure CLI** installed & authenticated (`az login`)  
- **Node.js & npm** (for building frontend)  
- **Python 3.9+ & pip** (for backend + migrations)  
- Local checkout of `shopkart-backend/` and `shopkart-frontend/`

---

## 🛠️ Deployment Steps

### 1\. Create a Resource Group  
```bash
export AZ_RG=shopkart-rg
export AZ_REGION=eastus
az group create --name $AZ_RG --location $AZ_REGION
```

### 2. Provision Data Services
####1.PostgreSQL
export AZ_PG_SERVER=shopkart-pg
export AZ_PG_ADMIN=pgadmin
export AZ_PG_PWD="<YourStrong!Passw0rd>"
```bash
az postgres server create \
  --resource-group $AZ_RG \
  --name $AZ_PG_SERVER \
  --location $AZ_REGION \
  --admin-user $AZ_PG_ADMIN \
  --admin-password $AZ_PG_PWD \
  --sku-name B_Gen5_1 \
  --version 13

az postgres server firewall-rule create \
  --resource-group $AZ_RG \
  --server $AZ_PG_SERVER \
  --name AllowAzureApps \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

#### 2. Redis Cache
```bash
export AZ_REDIS=shopkart-redis
az redis create \
  --resource-group $AZ_RG \
  --name $AZ_REDIS \
  --location $AZ_REGION \
  --sku Basic \
  --vm-size C0
```

### 3. Storage Account (Static Website)
```bash
export AZ_STORAGE=shopkartstatic
az storage account create \
  --resource-group $AZ_RG \
  --name $AZ_STORAGE \
  --location $AZ_REGION \
  --sku Standard_LRS \
  --kind StorageV2

az storage blob service-properties update \
  --account-name $AZ_STORAGE \
  --static-website \
  --index-document index.html \
  --404-document index.html
```

## 3. Deploy the Backend (Flask)
#### 1. Create Plan & Web App
```bash
export AZ_PLAN=shopkart-plan
export AZ_WEB=shopkart-api

az appservice plan create \
  --resource-group $AZ_RG \
  --name $AZ_PLAN \
  --is-linux \
  --sku B1

az webapp create \
  --resource-group $AZ_RG \
  --plan $AZ_PLAN \
  --name $AZ_WEB \
  --runtime "PYTHON|3.9"
```

#### 2. Configure Settings
```bash
# Postgres connection string
export PG_CONN="postgresql://${AZ_PG_ADMIN}:${AZ_PG_PWD}@${AZ_PG_SERVER}.postgres.database.azure.com/shopkart"
az webapp config connection-string set \
  --resource-group $AZ_RG \
  --name $AZ_WEB \
  --settings DATABASE_URL=$PG_CONN \
  --connection-string-type PostgreSQL

# JWT_SECRET, Redis URL
export REDIS_HOST=$(az redis show -g $AZ_RG -n $AZ_REDIS --query hostName -o tsv)
export REDIS_KEY=$(az redis show -g $AZ_RG -n $AZ_REDIS --query accessKeys.primaryKey -o tsv)

az webapp config appsettings set \
  --resource-group $AZ_RG \
  --name $AZ_WEB \
  --settings \
    JWT_SECRET="<your-jwt-secret>" \
    REDIS_URL="redis://:$REDIS_KEY@$REDIS_HOST:6379/0"
```

#### 3. Deploy & Migrate
```bash
# Zip and push
cd shopkart-backend
zip -r ../backend.zip .
az webapp deployment source config-zip \
  --resource-group $AZ_RG \
  --name $AZ_WEB \
  --src ../backend.zip

# Run migrations
az webapp ssh \
  --resource-group $AZ_RG \
  --name $AZ_WEB \
  --command "flask db upgrade"
```
## 4. Deploy the Frontend (React)
#### 1. Build
```bash
cd ../shopkart-frontend
npm install
npm run build
```

#### 2. Upload
```bash
az storage blob upload-batch \
  --account-name $AZ_STORAGE \
  --destination \$web \
  --source build
```

##  5. (Optional) Front with CDN
``` bash
export AZ_CDN=shopkart-cdn
az cdn profile create \
  --name $AZ_CDN \
  --resource-group $AZ_RG \
  --sku Standard_Microsoft

az cdn endpoint create \
  --name shopkart-endpoint \
  --profile-name $AZ_CDN \
  --resource-group $AZ_RG \
  --origin "${AZ_STORAGE}.z13.web.core.windows.net"
```

## 6. Monitoring & Alerts
#### 1. App Insights
```bash
export AZ_AI=shopkart-ai
az monitor app-insights component create \
  --app $AZ_AI \
  --location $AZ_REGION \
  --resource-group $AZ_RG \
  --kind web

az webapp config appsettings set \
  --resource-group $AZ_RG \
  --name $AZ_WEB \
  --settings APPINSIGHTS_INSTRUMENTATIONKEY=$(az monitor app-insights component show --app $AZ_AI --resource-group $AZ_RG --query instrumentationKey -o tsv)
```

#### 2. Log Stream & Alerts via Azure Portal under your Web App & App Insights blades.


## 🧪 Verification
### Frontend: browse to
```bash
https://<your-cdn>.azureedge.net (or https://<AZ_STORAGE>.z13.web.core.windows.net)
```
– Should load SPA, login, browse products, add to cart etc.

### Backend: curl
```bash

curl https://$AZ_WEB.azurewebsites.net/api/products
```
– Should return JSON product pages.

