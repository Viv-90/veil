# Contributing to Veil

## Setup

1. Clone the repository
2. Install dependencies:
   - Frontend: `cd frontend/website && npm install`
   - SDK: `cd sdk && npm install`
3. Copy the environment file:
   ```bash
   cp frontend/.env.example frontend/.env
   ```
4. Update the values in `frontend/.env` with your contract IDs (see [TESTNET_SETUP.md](./TESTNET_SETUP.md) for deployment instructions)
5. Run the development server:
   ```bash
   cd frontend/website && npm run dev
   ```

## Testing

Run tests with `npm test` in the respective package directories.