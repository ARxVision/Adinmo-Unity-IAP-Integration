# Adinmo Unity IAP Integration

Official integration guides for AdinmoPlugin with Unity In-App Purchasing (IAP).

## 📋 Quick Start

Choose your Unity IAP version:

- **[Unity IAP v4 Integration Guide](unity-iap-v4-integration.md)** - For Unity IAP v4.x.x
- **[Unity IAP v5 Integration Guide](unity-iap-v5-integration.md)** - For Unity IAP v5.x.x

## 🎯 What You'll Get

- ✅ **2-step integration** for existing Unity IAP projects
- ✅ **Complete code examples** ready to copy-paste
- ✅ **Automatic purchase tracking** to Adinmo analytics
- ✅ **Professional error handling** and troubleshooting

## 📋 Requirements

- Unity 2021.3 LTS or newer
- AdinmoPlugin v3.14 or above
- Unity IAP (UPM) v4.x.x or v5.x.x
- Download from [Adinmo Portal](https://www.adinmo.com/)

## 🚀 Integration Overview

Both integrations maintain your existing purchase logic while adding seamless Adinmo tracking:

### Unity IAP v4
```csharp
// Just change one line:
UnityPurchasingAdInMoExtensions.InitializeWithAdInMo(this, builder);
```

### Unity IAP v5
```csharp
// Use the new controller:
AdinmoStoreController m_StoreController = new AdinmoStoreController();
```

## 📁 Repository Contents

- [`unity-iap-v4-integration.md`](unity-iap-v4-integration.md) - Unity IAP v4 integration guide
- [`unity-iap-v5-integration.md`](unity-iap-v5-integration.md) - Unity IAP v5 integration guide

## 💡 Support

- 📖 **Documentation**: Browse the integration guides above
- 🐛 **Issues**: Report problems via GitHub Issues
- 🌐 **Adinmo Portal**: [https://www.adinmo.com/](https://www.adinmo.com/)

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.