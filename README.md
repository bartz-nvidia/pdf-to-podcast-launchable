<h2><img align="center" src="https://github.com/user-attachments/assets/cbe0d62f-c856-4e0b-b3ee-6184b7c4d96f">NVIDIA AI Blueprint: PDF to Podcast</h2>

## Overview

This NVIDIA AI blueprint shows developers how to build a microservice that transforms PDFs into engaging audio content. Built on NVIDIA NIM, this blueprint can run securely on a private network, delivering actionable insight without sharing sensitive data.

<img width="1021" alt="Screenshot 2024-12-30 at 8 43 43 PM" src="https://github.com/user-attachments/assets/604d0b4d-664f-4089-a30d-0431ff35aece" />

[mermaid diagram](docs/README.md)

## Quick Start Guide

1. **Set environment variables**
   
   ```bash
   # Create .env file with required variables
   echo "ELEVENLABS_API_KEY=your_key" > .env
   echo "NVIDIA_API_KEY=your_key" >> .env
   echo "MAX_CONCURRENT_REQUESTS=1" >> .env
   ```

   > **Note:** the NVIDIA Eleven Labs API key used in this example can handle concurrent requests. For local development, set `MAX_CONCURRENT_REQUESTS=1` to avoid rate-limiting issues. Generate your own API key for free [here](https://elevenlabs.io/).

2. **Install dependencies**

   We use [UV](https://pypi.org/project/uv/) to manage Python dependencies.
   
   ```bash
   make uv
   ```

    This will:
   - Install UV if not present
   - Create virtual environment
   - Install project dependencies

   If you open up a new terminal window and want to quickly re-use the same environment, you can run `make uv` again.

4. **Start the development Server**

   ```bash
   make all-services
   ```
   
   > **Note:** The first time you run `make all-services`, the `docling` service may take 10-15 minutes to pull and build. Subsequent runs will be much faster.

   This command will:
   - Verify environment variables are set
   - Create necessary directories
   - Start all services using Docker Compose in `--build` mode. 


  > **Note:** Set `DETACH=1` to run the services in detached mode to continue using your terminal while the services are running.

5. **Generate the Podcast**:

   ```bash
   source .venv/bin/activate
   python tests/test.py --target <pdf1.pdf> --context <pdf2.pdf>
   ```

   By default, this command will generate a 2-person podcast. Add the `--monologue` flag to generate a 1-person podcast.

## Customization

1. **Host the PDF service on a separate machine**

This blueprint uses [docling](https://github.com/DS4SD/docling) as the default PDF extraction service.

To run the PDF extraction service on a separate machine, add the following to your `.env` file: 
```bash
echo "MODEL_API_URL=<pdf-model-service-url" >> .env
```
The `make model-dev` target will let you spin up only the docling service.  

2. **Use a Self-hosted NIM** 

By default this blueprint uses an ensemble of 3 LLMS to generate podcasts. The example uses the LLama 3.1-70B NIM for balanced performance and accuracy. To use a different model, update the `models.json` file with the desired model. The default `models.json` calls an NVIDIA-hosted NIM. Feel free to use it as you develop locally. When you deploy, please use our NIM API Catalog endpoints.

3. **Change the Default Models and GPU Assignments**

It is easy to swap out different pieces of the stack to optimize GPU usage for available hardware. For example, minimize GPU usage by swapping in the smaller LLama 3.1-8B NIM and disabling GPU usage for `docling` in `docker-compose.yaml`.

4. **Enable Tracing**
   
We expose a Jaeger instance at `http://localhost:16686/` for tracing. This is useful for debugging and monitoring the system.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests: `python tests/test.py <pdf1> <pdf2>`
5. Run linting: `make ruff`
6. Submit a pull request

### Code Quality
The project uses `ruff` for linting and formatting. You must run `make ruff` before your PR can be merged:

```bash
make ruff  # Runs both lint and format
```
### CI/CD
We use GitHub Actions for CI/CD. We run the following actions:

- `ruff`: Runs linting and formatting
- `pr-test`: Runs an end-to-end podcast test on the PR
- `build-and-push`: Builds and pushes a new container image to the remote repo. This is used to update production deployments

## Security Considerations

**Important**: This setup uses HTTP and is not intended for production deployments. For production deployments, consider implementing the following security measures:

- Add SSL/TLS encryption by either:
  - Configuring uvicorn with SSL certificates
  - Setting up a reverse proxy (like Nginx) to handle SSL termination
- Implement proper certificate management
- Configure appropriate security headers
- Follow other web security best practices
