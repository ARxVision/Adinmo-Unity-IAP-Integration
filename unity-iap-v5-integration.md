# AdinmoPlugin Unity IAP v5 Integration Guide

## Requirements
- **Unity 2021.3 LTS or newer**
- **AdinmoPlugin v3.14 or above**
- **Unity IAP (UPM) v5.x.x**
- Download the AdinmoPlugin from the [Adinmo Portal](https://www.adinmo.com/)

## Installation Prerequisites

### 1. Install Unity IAP v5
1. **Open Package Manager**: `Window` → `Package Manager`
2. **Select Unity Registry** from the dropdown (top-left)
3. **Search for "In App Purchasing"**
4. **Install version 5.x.x** (latest stable)

### 2. Install AdinmoPlugin
1. **Download** the AdinmoPlugin package from [Adinmo Portal](https://www.adinmo.com/)
2. **Import** the `.unitypackage` file: `Assets` → `Import Package` → `Custom Package`
3. **Verify installation**: Check that `AdinmoPlugin` folder appears in your Assets
4. **Confirm version**: Ensure it's v3.14 or above in the package documentation

## Integration Guide

### For Existing Unity IAP v5 Users (Recommended)

If you already have Unity IAP v5 working, this is a simple 2-step upgrade:

#### Step 1: Add Compiler Symbol
1. **Open Player Settings**: `Edit` → `Project Settings` → `Player`
2. **Find Script Compilation section** (scroll down)
3. **Add to Scripting Define Symbols**: `ADINMO_UNITY_STORE_V5`
4. **Click Apply** and wait for recompilation

**Navigation Path:**
```
Edit → Project Settings → Player → Script Compilation → Scripting Define Symbols
Add: ADINMO_UNITY_STORE_V5
```

#### Step 2: Replace Your Unity IAP Controller
Replace your existing Unity IAP controller with `AdinmoStoreController`:

**Before:**
```csharp
StoreController m_StoreController = new StoreController();
// or traditional Unity IAP initialization
```

**After:**
```csharp
AdinmoStoreController m_StoreController = new AdinmoStoreController();
```

**That's it!** Your existing purchase handling logic can be adapted to the new event-based system.

## Complete Implementation Example

### Required Imports
```csharp
using UnityEngine;
using System.Threading.Tasks;
using System.Linq;
using System.Collections.Generic;
using UnityEngine.Purchasing;
using UnityEngine.Purchasing.Extension;
using Adinmo; // Add this for AdinmoStoreController
```

### Full Implementation
```csharp
public class MyIAPManager : MonoBehaviour
{
    private AdinmoStoreController m_StoreController;
    
    async void Start()
    {
        await InitializePurchasing();
    }
    
    private async Task InitializePurchasing()
    {
        if (m_StoreController != null)
            return;

        m_StoreController = new AdinmoStoreController();

        // Subscribe to purchase events
        m_StoreController.OnPurchasePending += OnPurchasePending;
        m_StoreController.OnPurchaseConfirmed += OnPurchaseConfirmed;

        try
        {
            // Connect to store services
            await m_StoreController.Connect();
            Debug.Log("Unity IAP v5 initialization successful - Adinmo integration active");
            
            // Fetch your products
            FetchProducts();
        }
        catch (System.Exception e)
        {
            Debug.LogError($"Store connection failed: {e}");
            ShowInitializationError();
        }
    }
    
    private void FetchProducts()
    {
        var products = new List<ProductDefinition>
        {
            new ProductDefinition("premium_upgrade", ProductType.NonConsumable),
            new ProductDefinition("gold_coins_100", ProductType.Consumable),
            new ProductDefinition("monthly_subscription", ProductType.Subscription)
        };
        
        m_StoreController.FetchProducts(products);
        Debug.Log("Products fetched and ready for purchase");
    }

    #region Purchase Event Handlers

    private void OnPurchasePending(PendingOrder order)
    {
        Debug.Log("Purchase pending - processing...");
        
        // Reward the player BEFORE confirming purchase
        var productId = order.CartOrdered.Items().First().Product.definition.id;
        ProcessPurchaseReward(productId);
        
        // Confirm the purchase (required)
        m_StoreController.ConfirmPurchase(order);
    }

    private void OnPurchaseConfirmed(Order order)
    {
        switch (order)
        {
            case FailedOrder failedOrder:
                var failedProduct = failedOrder.CartOrdered.Items().First().Product.definition.id;
                Debug.LogError($"Purchase failed: {failedProduct}, {failedOrder.FailureReason}");
                ShowPurchaseError(failedProduct, failedOrder.FailureReason.ToString());
                break;
                
            case ConfirmedOrder confirmedOrder:
                var confirmedProduct = order.CartOrdered.Items().First().Product.definition.id;
                Debug.Log($"Purchase confirmed: {confirmedProduct}");
                ShowPurchaseSuccess(confirmedProduct);
                break;
        }
    }

    #endregion

    #region Purchase Processing

    private void ProcessPurchaseReward(string productId)
    {
        Debug.Log($"Processing purchase reward: {productId}");
        
        switch (productId)
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
                Debug.LogWarning($"Unknown product: {productId}");
                break;
        }
        
        // Adinmo automatically tracks this purchase in the background
    }

    #endregion

    #region Public Purchase Methods
    
    public void BuyPremiumUpgrade()
    {
        BuyProduct("premium_upgrade");
    }
    
    public void BuyGoldCoins()
    {
        BuyProduct("gold_coins_100");
    }
    
    public void BuySubscription()
    {
        BuyProduct("monthly_subscription");
    }
    
    public void BuyProduct(string productId)
    {
        if (m_StoreController == null)
        {
            Debug.LogError("Store controller not initialized");
            return;
        }

        var product = m_StoreController.GetProducts().FirstOrDefault(p => p.definition.id == productId);
        
        if (product != null)
        {
            Debug.Log($"Initiating purchase: {productId}");
            m_StoreController.PurchaseProduct(product);
        }
        else
        {
            Debug.LogError($"Product not found: {productId}");
        }
    }

    public void RestorePurchases()
    {
        if (m_StoreController != null)
        {
            m_StoreController.RestoreTransactions(OnTransactionsRestored);
        }
    }
    
    private void OnTransactionsRestored(bool success, string error)
    {
        if (success)
        {
            Debug.Log("Transactions restored successfully");
        }
        else
        {
            Debug.LogError($"Transaction restoration failed: {error}");
        }
    }

    #endregion

    #region Game Logic (Your Implementation)
    
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
    
    private void ShowInitializationError()
    {
        // Your UI logic for showing initialization errors
        Debug.LogError("Failed to initialize store - check internet connection and store setup");
    }
    
    private void ShowPurchaseSuccess(string productId)
    {
        // Your UI logic for showing purchase success
        Debug.Log($"Purchase successful: {productId}");
    }
    
    private void ShowPurchaseError(string productId, string reason)
    {
        // Your UI logic for showing purchase errors
        Debug.LogError($"Purchase failed for {productId}: {reason}");
    }

    #endregion

    #region Helper Methods
    
    public bool IsStoreInitialized()
    {
        return m_StoreController != null;
    }
    
    public string GetProductPrice(string productId)
    {
        if (m_StoreController == null) return "N/A";
        
        var product = m_StoreController.GetProducts().FirstOrDefault(p => p.definition.id == productId);
        return product?.metadata.localizedPriceString ?? "N/A";
    }
    
    public bool IsProductOwned(string productId)
    {
        // For non-consumable products, check if already purchased
        return AdinmoStoreController.CheckAlreadyPurchased(productId).alreadyPurchased;
    }

    #endregion
}
```

## Critical: Product ID Matching

Your Unity IAP product IDs **must exactly match** your Adinmo campaign SKUs:

**Unity Code:**
```csharp
new ProductDefinition("premium_upgrade", ProductType.NonConsumable);
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

## Key Benefits of Unity IAP v5 + Adinmo

✅ **Event-Based Architecture**: Clean `OnPurchasePending` and `OnPurchaseConfirmed` events  
✅ **Async/Await Support**: Modern async patterns with `await Connect()`  
✅ **Enhanced Error Handling**: Detailed `FailedOrder` information  
✅ **Automatic Purchase Tracking**: Built-in Adinmo analytics integration  
✅ **Static Helper Methods**: `CheckAlreadyPurchased()`, `GetPrice()`, `PurchaseItem()`

## Troubleshooting

### Common Issues

#### 1. Compiler Symbol Not Applied
**Symptom**: `AdinmoStoreController` not found
**Solution**: 
- Add `ADINMO_UNITY_STORE_V5` to Scripting Define Symbols
- **Restart Unity Editor** after adding the symbol
- Wait for full recompilation

#### 2. SKU Mismatch
**Symptom**: Purchases work but don't appear in Adinmo dashboard
**Solution**:
- Double-check product ID spelling (case-sensitive)
- Verify SKU configuration in Adinmo portal
- Check for hidden characters or extra spaces

#### 3. Purchase Not Completing
**Symptom**: `OnPurchasePending` called but purchase doesn't finish
**Solution**:
- **Always call `ConfirmPurchase()`** in `OnPurchasePending`
- Ensure you reward the player before confirming
- Check for exceptions in the purchase processing logic

#### 4. Store Connection Issues
**Symptom**: `Connect()` fails or times out
**Solution**:
- Verify internet connection
- Check Unity Services configuration
- Ensure proper store setup (Google Play Console/App Store Connect)

---

This integration provides a modern, event-driven approach to Unity IAP while seamlessly adding Adinmo analytics tracking to your purchase flow.