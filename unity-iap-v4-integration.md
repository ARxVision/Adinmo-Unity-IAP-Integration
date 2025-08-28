# AdinmoPlugin Unity IAP v4 Integration Guide

## Requirements
- **Unity 2021.3 LTS or newer**
- **AdinmoPlugin v3.14 or above**
- **Unity IAP (UPM) v4.x.x**
- Download the AdinmoPlugin from the [Adinmo Portal](https://www.adinmo.com/)

## Installation Prerequisites

### 1. Install Unity IAP v4
1. **Open Package Manager**: `Window` → `Package Manager`
2. **Select Unity Registry** from the dropdown (top-left)
3. **Search for "In App Purchasing"**
4. **Install version 4.x.x** (latest stable)

### 2. Install AdinmoPlugin
1. **Download** the AdinmoPlugin package from [Adinmo Portal](https://www.adinmo.com/)
2. **Import** the `.unitypackage` file: `Assets` → `Import Package` → `Custom Package`
3. **Verify installation**: Check that `AdinmoPlugin` folder appears in your Assets
4. **Confirm version**: Ensure it's v3.14 or above in the package documentation

## Integration Guide

### For Existing Unity IAP v4 Users (Recommended)

If you already have Unity IAP v4 working, this is a simple 2-step upgrade:

#### Step 1: Add Compiler Symbol
1. **Open Player Settings**: `Edit` → `Project Settings` → `Player`
2. **Find Script Compilation section** (scroll down)
3. **Add to Scripting Define Symbols**: `ADINMO_UNITY_STORE_V4`
4. **Click Apply** and wait for recompilation

**Navigation Path:**
```
Edit → Project Settings → Player → Script Compilation → Scripting Define Symbols
Add: ADINMO_UNITY_STORE_V4
```

#### Step 2: Replace Initialize Call
Change just one line in your existing code:

**Before:**
```csharp
UnityPurchasing.Initialize(this, builder);
```

**After:**
```csharp
UnityPurchasingAdInMoExtensions.InitializeWithAdInMo(this, builder);
```

**That's it!** Your existing purchase handling logic remains completely unchanged.

## Complete Implementation Example

### Required Imports
```csharp
using UnityEngine;
using UnityEngine.Purchasing;
using UnityEngine.Purchasing.Security;
using Adinmo.Unity.Extensions; // Add this for Adinmo integration
```

### Full Implementation
```csharp
public class MyIAPManager : MonoBehaviour, IStoreListener
{
    private IStoreController storeController;
    private IExtensionProvider extensionProvider;
    
    void Start()
    {
        InitializePurchasing();
    }
    
    private void InitializePurchasing()
    {
        if (IsInitialized())
            return;

        var builder = ConfigurationBuilder.Instance(StandardPurchasingModule.Instance());
        
        // Add your products
        builder.AddProduct("premium_upgrade", ProductType.NonConsumable);
        builder.AddProduct("gold_coins_100", ProductType.Consumable);
        builder.AddProduct("monthly_subscription", ProductType.Subscription);

        // Initialize with Adinmo integration
        UnityPurchasingAdInMoExtensions.InitializeWithAdInMo(this, builder);
        
        Debug.Log("Unity IAP initializing with Adinmo extensions...");
    }
    
    private bool IsInitialized()
    {
        return storeController != null && extensionProvider != null;
    }

    #region IStoreListener Implementation

    public void OnInitialized(IStoreController controller, IExtensionProvider extensions)
    {
        Debug.Log("Unity IAP initialization successful - Adinmo integration active");
        
        storeController = controller;
        extensionProvider = extensions;
        
        // Your existing initialization logic here
        DisplayAvailableProducts();
    }

    public void OnInitializeFailed(InitializationFailureReason error)
    {
        Debug.LogError($"Unity IAP initialization failed: {error}");
        
        switch (error)
        {
            case InitializationFailureReason.AppNotKnown:
                Debug.LogError("App not configured in store");
                break;
            case InitializationFailureReason.PurchasingUnavailable:
                Debug.LogError("Purchasing unavailable on this device");
                break;
            case InitializationFailureReason.NoProductsAvailable:
                Debug.LogError("No products available for purchase");
                break;
        }
        
        // Handle initialization failure in your UI
        ShowInitializationError(error);
    }

    public PurchaseProcessingResult ProcessPurchase(PurchaseEventArgs args)
    {
        Debug.Log($"Processing purchase: {args.purchasedProduct.definition.id}");
        
        // Your existing purchase handling logic
        switch (args.purchasedProduct.definition.id)
        {
            case "premium_upgrade":
                UnlockPremium();
                Debug.Log("Premium upgrade unlocked");
                break;
                
            case "gold_coins_100":
                AddCoins(100);
                Debug.Log("Added 100 gold coins");
                break;
                
            case "monthly_subscription":
                ActivateSubscription();
                Debug.Log("Monthly subscription activated");
                break;
                
            default:
                Debug.LogWarning($"Unknown product: {args.purchasedProduct.definition.id}");
                break;
        }
        
        // Adinmo automatically tracks this purchase in the background
        return PurchaseProcessingResult.Complete;
    }

    public void OnPurchaseFailed(Product product, PurchaseFailureReason failureReason)
    {
        Debug.LogError($"Purchase failed - Product: {product.definition.storeSpecificId}, Reason: {failureReason}");
        
        switch (failureReason)
        {
            case PurchaseFailureReason.BillingUnavailable:
                Debug.LogError("Billing unavailable");
                break;
            case PurchaseFailureReason.UserCancelled:
                Debug.Log("User cancelled purchase");
                break;
            case PurchaseFailureReason.PaymentDeclined:
                Debug.LogError("Payment declined");
                break;
            case PurchaseFailureReason.DuplicateTransaction:
                Debug.LogError("Duplicate transaction");
                break;
            case PurchaseFailureReason.ProductUnavailable:
                Debug.LogError($"Product unavailable: {product.definition.id}");
                break;
        }
        
        // Handle purchase failure in your UI
        ShowPurchaseError(product, failureReason);
    }

    #endregion

    #region Public Purchase Methods
    
    public void BuyPremiumUpgrade()
    {
        BuyProductID("premium_upgrade");
    }
    
    public void BuyGoldCoins()
    {
        BuyProductID("gold_coins_100");
    }
    
    public void BuySubscription()
    {
        BuyProductID("monthly_subscription");
    }
    
    private void BuyProductID(string productId)
    {
        if (!IsInitialized())
        {
            Debug.LogError("IAP not initialized");
            return;
        }

        Product product = storeController.products.WithID(productId);
        
        if (product != null && product.availableToPurchase)
        {
            Debug.Log($"Purchasing product: {product.definition.id}");
            storeController.InitiatePurchase(product);
        }
        else
        {
            Debug.LogError($"Product not available for purchase: {productId}");
        }
    }

    #endregion

    #region Game Logic (Your Implementation)
    
    private void DisplayAvailableProducts()
    {
        // Your UI logic to show available products
        foreach (Product product in storeController.products.all)
        {
            if (product.availableToPurchase)
            {
                Debug.Log($"Available: {product.definition.id} - {product.metadata.localizedPriceString}");
            }
        }
    }
    
    private void UnlockPremium()
    {
        // Your game logic for premium unlock
        PlayerPrefs.SetInt("PremiumUnlocked", 1);
    }
    
    private void AddCoins(int amount)
    {
        // Your game logic for adding coins
        int currentCoins = PlayerPrefs.GetInt("GoldCoins", 0);
        PlayerPrefs.SetInt("GoldCoins", currentCoins + amount);
    }
    
    private void ActivateSubscription()
    {
        // Your game logic for subscription activation
        PlayerPrefs.SetString("SubscriptionActive", System.DateTime.Now.ToString());
    }
    
    private void ShowInitializationError(InitializationFailureReason error)
    {
        // Your UI logic for showing initialization errors
    }
    
    private void ShowPurchaseError(Product product, PurchaseFailureReason reason)
    {
        // Your UI logic for showing purchase errors
    }

    #endregion
}
```

## Critical: Product ID Matching

Your Unity IAP product IDs **must exactly match** your Adinmo campaign SKUs:

**Unity Code:**
```csharp
builder.AddProduct("premium_upgrade", ProductType.NonConsumable);
// Product ID: "premium_upgrade"
```

**Adinmo Dashboard:**
```
Campaign → SKU ID: "premium_upgrade"  ← Must match exactly (case-sensitive)
```

**Common Matching Issues:**
- ❌ `"Premium_Upgrade"` vs `"premium_upgrade"` (case mismatch)
- ❌ `"premium-upgrade"` vs `"premium_upgrade"` (separator mismatch)
- ❌ `"com.yourcompany.premium_upgrade"` vs `"premium_upgrade"` (prefix mismatch)

## Troubleshooting

### Common Issues

#### 1. Compiler Symbol Not Applied
**Symptom**: `UnityPurchasingAdInMoExtensions` not found
**Solution**: 
- Add `ADINMO_UNITY_STORE_V4` to Scripting Define Symbols
- **Restart Unity Editor** after adding the symbol
- Wait for full recompilation

#### 2. SKU Mismatch
**Symptom**: Purchases work but don't appear in Adinmo dashboard
**Solution**:
- Double-check product ID spelling (case-sensitive)
- Verify SKU configuration in Adinmo portal
- Check for hidden characters or extra spaces

#### 3. Missing Dependencies
**Symptom**: Build errors or missing references
**Solution**:
- Ensure Unity IAP v4 is installed via Package Manager
- Verify AdinmoPlugin v3.14+ is imported
- Check that all required assemblies are referenced

---

This integration maintains all your existing Unity IAP logic while seamlessly adding Adinmo analytics tracking to your purchase flow.