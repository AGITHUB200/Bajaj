
/for main.py/

from fastapi import FastAPI, UploadFile, File
from fastapi.responses import JSONResponse
from PIL import Image
import pytesseract
import io
import re

app = FastAPI()

def extract_text_from_image(image):
    return pytesseract.image_to_string(image)

def parse_lab_tests(text):
    pattern = re.findall(r"([A-Za-z\s]+)\s+(\d+\.?\d*)\s+(\d+\.?\d*\s*-\s*\d+\.?\d*)", text)
    results = []
    for name, value, ref_range in pattern:
        try:
            val = float(value)
            low, high = map(float, ref_range.replace(" ", "").split("-"))
            results.append({
                "lab_test_name": name.strip(),
                "lab_test_value": val,
                "bio_reference_range": f"{low}-{high}",
                "lab_test_out_of_range": not (low <= val <= high)
            })
        except:
            continue
    return results

@app.post("/get-lab-tests")
async def get_lab_tests(file: UploadFile = File(...)):
    try:
        contents = await file.read()
        image = Image.open(io.BytesIO(contents))
        text = extract_text_from_image(image)
        lab_tests = parse_lab_tests(text)

        return {
            "is_success": True,
            "lab_test_data": lab_tests
        }
    except Exception as e:
        return {
            "is_success": False,
            "error": str(e)
        }






/for process_folder.py/
import os
from PIL import Image
import pytesseract
import re

# Path to your folder containing lab report images
folder_path = r'C:\Users\HP\Downloads\lab_reports_samples\__MACOSX\lbmaske'

def extract_text_from_image(image):
    return pytesseract.image_to_string(image)

def parse_lab_tests(text):
    pattern = re.findall(r"([A-Za-z\s]+)\s+(\d+\.?\d*)\s+(\d+\.?\d*\s*-\s*\d+\.?\d*)", text)
    results = []
    for name, value, ref_range in pattern:
        try:
            val = float(value)
            low, high = map(float, ref_range.replace(" ", "").split("-"))
            results.append({
                "lab_test_name": name.strip(),
                "lab_test_value": val,
                "bio_reference_range": f"{low}-{high}",
                "lab_test_out_of_range": not (low <= val <= high)
            })
        except:
            continue
    return results

def extract_lab_data_from_image(image):
    text = extract_text_from_image(image)
    return parse_lab_tests(text)

# Process each image in the folder
for filename in os.listdir(folder_path):
    if filename.lower().endswith(('.png', '.jpg', '.jpeg')):
        image_path = os.path.join(folder_path, filename)
        image = Image.open(image_path)
        result = extract_lab_data_from_image(image)
        print(f"Results for {filename}:\n", result)



# Bajaj
Repository for Bajaj Finserv
import os
from PIL import Image

folder_path = 'C:\Users\HP\Downloads\lab_reports_samples\__MACOSX\lbmaske'

for filename in os.listdir(folder_path):
    if filename.lower().endswith(('.png', '.jpg', '.jpeg')):
        image_path = os.path.join(folder_path, filename)
        image = Image.open(image_path)

        # Now you can process each image
        result = extract_lab_data_from_image(image)
        print(result)
pip install pytesseract pillow fastapi uvicorn
import pytesseract
from PIL import Image

def extract_text_from_image(image):
    return pytesseract.image_to_string(image)
import re

def parse_lab_tests(text):
    pattern = re.findall(r"([A-Za-z\s]+)\s+(\d+\.?\d*)\s+(\d+\.?\d*\s*-\s*\d+\.?\d*)", text)
    
    results = []
    for name, value, ref_range in pattern:
        try:
            val = float(value)
            low, high = map(float, ref_range.replace(" ", "").split("-"))
            results.append({
                "lab_test_name": name.strip(),
                "lab_test_value": val,
                "bio_reference_range": f"{low}-{high}",
                "lab_test_out_of_range": not (low <= val <= high)
            })
        except:
            continue  # skip malformed lines
    return results
from fastapi import FastAPI, UploadFile, File
from fastapi.responses import JSONResponse
import io

app = FastAPI()

@app.post("/get-lab-tests")
async def get_lab_tests(file: UploadFile = File(...)):
    try:
        contents = await file.read()
        image = Image.open(io.BytesIO(contents))

        text = extract_text_from_image(image)
        lab_tests = parse_lab_tests(text)

        return {
            "is_success": True,
            "lab_test_data": lab_tests
        }
    except Exception as e:
        return {
            "is_success": False,
            "error": str(e)
        }
uvicorn main:app --reload

/output will be of the format/
{
  "is_success": true,
  "lab_test_data": [
    {
      "lab_test_name": "Hemoglobin",
      "lab_test_value": 10.5,
      "bio_reference_range": "12.0-16.0",
      "lab_test_out_of_range": true
    }
  ]
}
