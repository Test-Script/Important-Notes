========================================================================================================
            An Azure App Service Plan starts incurring cost immediately after creation
========================================================================================================

1. App Service Plan :- Reserved Resources are allocated to us.

    Compute resources (VM instances)

    Reserved CPU & RAM capacity

    Underlying infrastructure

We need to pay for the resources those are reserved for the us under the App service plan, even it is not in use.

App Service Plan = Reserved VM capacity
Web App = Application running on that VM

2. Pricing Tier Matters

    Billing behavior depends on the SKU:

        - Free (F1)
        - Shared (D1)
        - Basic / Standard / Premium / Isolated

3. Scaling Impact

    Billing is based on:

    Price per instance × Number of instances × Running hours

    If you scale to 3 instances → you pay for 3 instances continuously.

4. What Stops Billing?

    Only these actions stop billing:

    - Delete the App Service Plan

    - Downgrade to Free tier

5. Important Exceptioin

    Azure Functions (Consumption Plan)

    You pay per execution (serverless model)
