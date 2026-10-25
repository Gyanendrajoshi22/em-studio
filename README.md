# em-studio
Android interface and cloud solver backend for a full-wave electromagnetic simulator

from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI(title="EM Studio API")


class MicrostripProject(BaseModel):
    trace_width_mm: float = Field(default=1.5, gt=0, le=50)
    trace_length_mm: float = Field(default=30.0, gt=0, le=500)
    substrate_height_mm: float = Field(default=1.6, gt=0, le=50)
    relative_permittivity: float = Field(default=4.4, ge=1, le=30)
    start_frequency_ghz: float = Field(default=1.0, gt=0, le=100)
    stop_frequency_ghz: float = Field(default=5.0, gt=0, le=100)
    points: int = Field(default=101, ge=11, le=1001)


@app.get("/")
def home():
    return {
        "app": "EM Studio",
        "status": "API starter",
        "solver_ready": False
    }


@app.get("/health")
def health():
    return {
        "api": "ok",
        "openems_ready": False,
        "simulation_tested": False
    }


@app.post("/projects/validate")
def validate(project: MicrostripProject):
    if project.stop_frequency_ghz <= project.start_frequency_ghz:
        return {
            "valid": False,
            "error": "Stop frequency must exceed start frequency"
        }

    return {
        "valid": True,
        "status": "input_validation_only",
        "message": "No electromagnetic simulation has been run.",
        "project": project.model_dump()
    }
