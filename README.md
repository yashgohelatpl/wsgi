Understood. You want to run your FastAPI application on Apache instead of using Uvicorn directly. To do this, we'll need to use WSGI (Web Server Gateway Interface) with Apache. Here's how we can modify your code to work with Apache:

```python
import os
import logging
from typing import List, Dict, Any
from pydantic import BaseModel, Field
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import HTMLResponse, FileResponse
from fastapi.staticfiles import StaticFiles
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.vectorstores import FAISS
from langchain.prompts import PromptTemplate
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings import HuggingFaceBgeEmbeddings
from langchain_groq import ChatGroq
from langchain.schema import Document as LangchainDocument
import nltk
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np
from functools import lru_cache
import asyncio
from contextlib import asynccontextmanager
import PyPDF2

# ... (rest of the imports and configurations remain the same)

# Initialize the FastAPI app
app = FastAPI()

# ... (rest of the code remains the same until the end)

# Remove the __main__ block with Uvicorn
# if __name__ == "__main__":
#     import uvicorn
#     uvicorn.run(app, host="0.0.0.0", port=8000)

# Instead, add this for WSGI compatibility
from fastapi.middleware.wsgi import WSGIMiddleware

# Wrap the FastAPI app with WSGIMiddleware
wsgi_app = WSGIMiddleware(app)

# This function will be used by mod_wsgi
def application(environ, start_response):
    return wsgi_app(environ, start_response)

```

Now that we've modified the code to work with WSGI, you'll need to set up Apache with mod_wsgi to serve your FastAPI application. Here are the steps to do this:

1. Install mod_wsgi for Apache:

   ```
   sudo apt-get install libapache2-mod-wsgi-py3
   ```

2. Create a WSGI file (e.g., `wsgi.py`) in your project directory:

   ```python
   from your_script_name import application
   ```

   Replace `your_script_name` with the actual name of your Python script.

3. Configure Apache to serve your FastAPI app. Create a new Apache configuration file or modify the existing one:

   ```apache
   <VirtualHost *:80>
       ServerName yourdomain.com
       DocumentRoot /path/to/your/project

       WSGIDaemonProcess yourapp python-path=/path/to/your/project:/path/to/your/venv/lib/python3.x/site-packages
       WSGIProcessGroup yourapp
       WSGIScriptAlias / /path/to/your/project/wsgi.py

       <Directory /path/to/your/project>
           Require all granted
       </Directory>
   </VirtualHost>
   ```

   Replace `yourdomain.com` with your actual domain or server IP, and adjust the paths accordingly.

4. Enable the site configuration and restart Apache:

   ```
   sudo a2ensite your_config_file
   sudo systemctl restart apache2
   ```

5. Make sure all the required Python packages are installed in your virtual environment or system-wide Python installation.

6. Ensure that Apache has the necessary permissions to read and execute your FastAPI application files.

By following these steps, your FastAPI application should now run on Apache using mod_wsgi instead of Uvicorn. This setup allows Apache to handle incoming requests and pass them to your FastAPI application through the WSGI interface.

Remember to adjust file paths, permissions, and Apache configurations according to your specific server setup and requirements. Also, make sure that all the necessary dependencies are installed in the Python environment that Apache's mod_wsgi is using.
