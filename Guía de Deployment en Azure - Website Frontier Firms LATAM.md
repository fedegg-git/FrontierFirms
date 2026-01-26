
# Guía de Deployment en Azure - Website Frontier Firms LATAM

## 🚀 **Opciones de Deployment (Recomendadas por Simplicidad)**

### **Opción 1: Azure Static Web Apps (RECOMENDADA - Más Simple)**
### **Opción 2: Azure App Service**
### **Opción 3: Azure Storage + CDN**
### **Opción 4: Azure Container Instances**

---

## 🎯 **OPCIÓN 1: Azure Static Web Apps (RECOMENDADA)**

### **¿Por qué esta opción?**
- ✅ **Gratis** para sitios pequeños/medianos
- ✅ **SSL automático** y dominio personalizado
- ✅ **CI/CD integrado** con GitHub
- ✅ **Global CDN** incluido
- ✅ **Perfecto para sitios estáticos** como el nuestro

### **Paso a Paso:**

#### **Paso 1: Preparar el Código**
```bash
# 1. Crear repositorio en GitHub
# Ve a github.com y crea un nuevo repositorio llamado "frontier-firms-latam"

# 2. Subir archivos al repositorio
git init
git add .
git commit -m "Initial commit - Frontier Firms LATAM website"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/frontier-firms-latam.git
git push -u origin main
```

#### **Paso 2: Crear Azure Static Web App**
1. **Ir al Portal de Azure**: https://portal.azure.com
2. **Crear Recurso**: Click en "Create a resource"
3. **Buscar**: "Static Web Apps"
4. **Configurar**:
   ```
   Subscription: [Tu suscripción]
   Resource Group: [Crear nuevo] "rg-frontier-firms-latam"
   Name: "swa-frontier-firms-latam"
   Plan type: Free
   Region: East US 2
   Source: GitHub
   ```

#### **Paso 3: Conectar con GitHub**
1. **Autorizar GitHub**: Click "Sign in with GitHub"
2. **Seleccionar Repositorio**: 
   - Organization: Tu usuario
   - Repository: frontier-firms-latam
   - Branch: main
3. **Build Details**:
   ```
   Build Presets: Custom
   App location: /website
   Api location: (dejar vacío)
   Output location: /website
   ```

#### **Paso 4: Deploy**
1. **Review + Create**: Verificar configuración
2. **Create**: Azure creará el recurso y configurará CI/CD
3. **Esperar**: El deployment inicial toma 2-3 minutos

#### **Paso 5: Verificar**
- **URL temporal**: Azure te dará una URL como `https://happy-sea-123456.azurestaticapps.net`
- **GitHub Actions**: Ve a tu repo → Actions para ver el progreso

---

## 🎯 **OPCIÓN 2: Azure App Service**

### **Cuándo usar esta opción:**
- Necesitas más control sobre el servidor
- Planeas agregar backend/APIs
- Quieres escalabilidad automática

### **Paso a Paso:**

#### **Paso 1: Crear App Service**
```bash
# Usando Azure CLI (instalar desde https://docs.microsoft.com/cli/azure/install-azure-cli)
az login
az group create --name rg-frontier-firms-latam --location eastus2
az appservice plan create --name asp-frontier-firms --resource-group rg-frontier-firms-latam --sku F1 --is-linux
az webapp create --name frontier-firms-latam --resource-group rg-frontier-firms-latam --plan asp-frontier-firms --runtime "NODE|18-lts"
```

#### **Paso 2: Configurar Deployment**
```bash
# Opción A: Desde repositorio local
az webapp deployment source config-local-git --name frontier-firms-latam --resource-group rg-frontier-firms-latam

# Opción B: Desde GitHub
az webapp deployment source config --name frontier-firms-latam --resource-group rg-frontier-firms-latam --repo-url https://github.com/TU-USUARIO/frontier-firms-latam --branch main --manual-integration
```

#### **Paso 3: Configurar para Sitio Estático**
1. **Portal Azure** → App Service → Configuration
2. **Agregar Setting**:
   ```
   Name: WEBSITE_NODE_DEFAULT_VERSION
   Value: 18.17.0
   ```
3. **Startup Command**: `npx serve website -s`

---

## 🎯 **OPCIÓN 3: Azure Storage + CDN (Más Económica)**

### **Cuándo usar:**
- Máximo rendimiento
- Costo mínimo
- Control total sobre CDN

### **Paso a Paso:**

#### **Paso 1: Crear Storage Account**
```bash
az storage account create \
  --name stfrontierfirmslatam \
  --resource-group rg-frontier-firms-latam \
  --location eastus2 \
  --sku Standard_LRS \
  --kind StorageV2
```

#### **Paso 2: Habilitar Static Website**
```bash
az storage blob service-properties update \
  --account-name stfrontierfirmslatam \
  --static-website \
  --404-document 404.html \
  --index-document index.html
```

#### **Paso 3: Subir Archivos**
```bash
# Obtener connection string
az storage account show-connection-string --name stfrontierfirmslatam --resource-group rg-frontier-firms-latam

# Subir archivos
az storage blob upload-batch \
  --account-name stfrontierfirmslatam \
  --destination '$web' \
  --source website/
```

#### **Paso 4: Configurar CDN (Opcional)**
```bash
az cdn profile create \
  --name cdn-frontier-firms \
  --resource-group rg-frontier-firms-latam \
  --sku Standard_Microsoft

az cdn endpoint create \
  --name frontier-firms-endpoint \
  --profile-name cdn-frontier-firms \
  --resource-group rg-frontier-firms-latam \
  --origin stfrontierfirmslatam.z13.web.core.windows.net
```

---

## 🎯 **OPCIÓN 4: Deployment Manual Rápido (Sin CLI)**

### **Para usuarios que prefieren la interfaz gráfica:**

#### **Paso 1: Portal Azure**
1. **Ir a**: https://portal.azure.com
2. **Create Resource** → "Static Web Apps"
3. **Configurar**:
   - Name: `frontier-firms-latam`
   - Region: `East US 2`
   - Source: `Other` (para upload manual)

#### **Paso 2: Upload Manual**
1. **Después de crear** → ir al recurso
2. **Browse** → encontrar opción de upload
3. **Subir archivos** desde la carpeta `website/`

---

## 🔧 **Configuraciones Adicionales Recomendadas**

### **1. Dominio Personalizado**
```bash
# Después del deployment
az staticwebapp hostname set \
  --name frontier-firms-latam \
  --resource-group rg-frontier-firms-latam \
  --hostname www.frontierfirms-latam.com
```

### **2. Variables de Entorno**
```json
{
  "ANALYTICS_ID": "GA-XXXXXXXXX",
  "API_ENDPOINT": "https://api.frontierfirms-latam.com",
  "ENVIRONMENT": "production"
}
```

### **3. Headers de Seguridad**
```json
{
  "routes": [
    {
      "route": "/*",
      "headers": {
        "X-Frame-Options": "DENY",
        "X-Content-Type-Options": "nosniff",
        "X-XSS-Protection": "1; mode=block",
        "Strict-Transport-Security": "max-age=31536000; includeSubDomains"
      }
    }
  ]
}
```

---

## 📊 **Comparación de Opciones**

| Característica | Static Web Apps | App Service | Storage + CDN | Manual |
|----------------|-----------------|-------------|---------------|---------|
| **Costo** | Gratis/Muy bajo | $13-50/mes | $1-5/mes | Gratis/Muy bajo |
| **Simplicidad** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| **Performance** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Escalabilidad** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **CI/CD** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ |
| **SSL/HTTPS** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 🚨 **Troubleshooting Común**

### **Error: "Build failed"**
```bash
# Verificar estructura de archivos
website/
├── index.html
├── styles.css
├── script.js
└── README.md
```

### **Error: "404 Not Found"**
- Verificar que `index.html` esté en la raíz de `/website`
- Configurar `index.html` como documento por defecto

### **Error: "CORS Issues"**
```javascript
// Agregar en script.js si usas APIs externas
const headers = {
  'Content-Type': 'application/json',
  'Access-Control-Allow-Origin': '*'
};
```

---

## 🎯 **Recomendación Final**

**Para tu caso específico, recomiendo OPCIÓN 1 (Azure Static Web Apps):**

1. **Más simple** de configurar
2. **Gratis** para tu volumen de tráfico
3. **CI/CD automático** con GitHub
4. **SSL y CDN** incluidos
5. **Perfecto** para sitios estáticos como el tuyo

### **Tiempo estimado de deployment: 10-15 minutos**

¿Te gustaría que te ayude con alguna de estas opciones específicamente o tienes alguna pregunta sobre el proceso?
