## Dependencies

## How to Run Locally

1. **Create a `.env` file to store secrets**
   Create a `.env` file in the `root` folder and add:
```
   OPENWEATHER_API_KEY=
```
 
2. **Create a virtual environment**
```
   python -m venv venv
```
 
3. **Activate the virtual environment**
   Windows:
```
   venv\Scripts\Activate.ps1
```
 
   Ubuntu / macOS:
```
   source venv/bin/activate
```
 
4. **Install dependencies**

   To run the service:
   ```
   pip install -r requirements.txt
   ```

   For development:
   ```
   pip install -r requirements-dev.txt
   ```
 
5. **Run application**
```
   uvicorn app.main:app --reload
```


6. **Run tests**
```
   pytest -v
```
