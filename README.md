# Panda Lite API Testing

**Course Work – API Testing**

This repository contains the API testing suite for the **Panda Lite** project, created as part of a coursework assignment. The tests are written using [Postman/Newman](https://www.postman.com/) (or specify actual framework) and can be executed locally to verify the correctness and reliability of the Panda Lite REST API.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Running the Tests](#running-the-tests)
- [Test Structure](#test-structure)
- [Project Structure](#project-structure)
- [Adding to Your CV](#adding-to-your-cv)
- [License](#license)
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Running the Tests](#running-the-tests)
- [Test Structure](#test-structure)
- [Adding to Your CV](#adding-to-your-cv)
- [License](#license)

## Overview
The test suite covers:
- Endpoint existence and response status codes
- Input validation and error handling
- Data integrity checks for CRUD operations
- Performance benchmarks (optional)

## Prerequisites
- Node.js (v14 or later)
- npm (comes with Node)
- Newman (`npm install -g newman`) – command‑line runner for Postman collections

## Setup
1. Clone the repository:
   ```
   git clone <repository‑url>
   cd Panda_lite_API_Testing
   ```
2. Install any required npm packages (if any):
   ```
   npm install
   ```
3. Ensure the Panda Lite API server is running locally (default `http://localhost:3000`).

## Running the Tests
Execute the following command to run the full collection:
```bash
newman run collections/PandaLite.postman_collection.json -e environments/local.env.json
```
You can also run individual folders or specific request tests using Newman flags.

## Test Structure
- `collections/` – Postman collection files.
- `environments/` – Environment files with variables like `{{baseUrl}}`.
- `scripts/` – Helper scripts for data setup/teardown.

## Project Structure

- `collections/` – Postman collection files.
- `environments/` – Environment files with variables like `{{baseUrl}}`.
- `scripts/` – Helper scripts for data setup/teardown.
- `Course work details.md` – Documentation of coursework requirements.
- `documentation.md` – Additional project documentation.
- `Panda Lite Api V2.postman_collection.json` – Main Postman collection file.
- `Panda-Lite Environment.postman_environment.json` – Environment configuration.
- `README.md` – This file.
- `SystemTestingReport.md` – Report of system testing results.



## License
This coursework project is provided for educational purposes and is licensed under the MIT License.

