# Ocean Monitoring Lakehouse

## About the Project

This project is an end-to-end streaming data pipeline on Databricks that simulates, ingests, cleans, and aggregates measurements from a network of five equatorial Pacific Ocean monitoring buoys. It demonstrates the medallion architecture (landing, bronze, silver, gold) implemented with Lakeflow Spark Declarative Pipelines, Delta Lake, and Unity Catalog.

The project covers:

- Generating realistic synthetic sensor data with a mean-reverting stochastic model.
- Incremental streaming ingestion of six independent sensor streams.
- Data enrichment with static buoy metadata and data quality expectations.
- Windowed streaming aggregations with watermarking for late-arriving data.

## Architecture

```
00_data_generator  -->  00_landing (UC Volume, Delta files)
                              |
                              v
                        01_bronze (raw streaming tables)
                              |
                              v
                        02_silver (cleaned, enriched streaming tables)
                              |
                              v
                        03_gold (24-hour aggregated buoy statistics)
```

All objects live in the `ocean_monitoring` Unity Catalog catalog, with one schema per layer.

| Layer | Location | Description |
|---|---|---|
| Landing | `/Volumes/ocean_monitoring/00_landing/streaming/<parameter>` | Raw Delta files appended by the data generator, one folder per parameter. |
| Bronze | `ocean_monitoring.01_bronze` | Streaming tables `water_temperature`, `salinity`, `current_speed`, `wind_speed`, `wave_height`, `wave_period` ingested as-is from the landing volume. |
| Silver | `ocean_monitoring.02_silver` | Static `buoy_metadata` table plus one streaming table per parameter, joined with buoy metadata, with standardized column names and data quality expectations. |
| Gold | `ocean_monitoring.03_gold` | `buoy_stats` table with average and maximum values of all six parameters per buoy and 24-hour simulated window. |

## Repository Structure

| Path | Description |
|---|---|
| `00_data_generator` | Notebook that simulates the five buoys and continuously writes synthetic measurements to the landing volume. |
| `01_bronze_processing` | Pipeline source defining the bronze streaming tables. |
| `02_silver_processing` | Pipeline source defining buoy metadata and the silver streaming tables with expectations. |
| `03_gold_processing` | Pipeline source defining the gold aggregated statistics table. |
| `README.md` | Project and domain documentation. |

## Processing Details

### Data Generator

- Runs six generator threads in parallel, one per parameter.
- Each batch produces one row per buoy (5 rows) and is appended every 60 seconds; each batch advances simulation time by one hour.
- Values evolve with a mean-reverting random walk clipped to the parameter range. Temperature and wind speed follow a daily cycle based on the buoy's local solar time, and wind speed includes occasional gusts.
- Each row contains `buoy_id`, `latitude`, `longitude`, `simulation_timestamp`, `ingestion_timestamp`, and the parameter value. `ingestion_timestamp` is shifted back by a random latency of up to 60 seconds to emulate transmission delay.

### Bronze Layer

Reads each landing folder as a Delta stream and stores the records without transformation.

### Silver Layer

- `buoy_metadata` holds static attributes for each buoy: name, manufacturer, model, production year, type, and power source.
- Each parameter stream is left-joined with `buoy_metadata` on `buoy_id` and the value column is renamed (for example, `temperature_c` to `water_temperature`).
- Data quality expectations:
  - Rows with a null `ingestion_timestamp` are dropped.
  - Values outside the expected range are flagged (recorded in pipeline metrics, not dropped).

### Gold Layer

- Applies a 2-minute watermark on `ingestion_timestamp` to handle late data.
- Aggregates each parameter in 24-minute tumbling windows per buoy (24 simulated hours).
- Computes average (rounded to 2 decimals) and maximum values, then joins all six parameters on `buoy_id`, `window_start`, and `window_end` into `buoy_stats`.

## Getting Started

1. Create the `ocean_monitoring` catalog with schemas `00_landing`, `01_bronze`, `02_silver`, and `03_gold`, and a `streaming` volume in `00_landing`.
2. Run the `00_data_generator` notebook. It runs continuously until stopped.
3. Create a Lakeflow Spark Declarative Pipeline with `01_bronze_processing`, `02_silver_processing`, and `03_gold_processing` as source code and `ocean_monitoring` as the default catalog.
4. Start the pipeline in triggered or continuous mode and query `ocean_monitoring.03_gold.buoy_stats` for results.

# Ocean Monitoring Parameters

## Overview

The simulated ocean monitoring system consists of five autonomous monitoring buoys deployed along the Equator in the Pacific Ocean. Each buoy is equipped with sensors to measure six oceanographic and meteorological parameters.

The system monitors normal ocean conditions using accelerated time simulation, generating synthetic measurements every **1 minute**, where one real minute represents one simulated hour. The selected parameters provide information about ocean temperature, salinity, circulation, wind, and wave conditions.

## Sensor Configuration

Each monitoring buoy is equipped with the following sensors:

| Sensor | Placement | Measurements |
|---|---|---|
| Thermistor | 1 m below the sea surface | Water temperature |
| Conductivity sensor | 1 m below the sea surface | Salinity |
| Acoustic Doppler Current Profiler (ADCP) | Attached to the underwater mooring, measuring at approximately 10 m depth | Ocean current speed |
| Anemometer | Approximately 4 m above the sea surface | Wind speed |
| Accelerometer-based wave sensor | Integrated into the floating buoy | Significant wave height and wave period |

## Monitored Parameters

### 1. Sea Surface Temperature (SST)

Measures the temperature of seawater approximately 1 m below the surface. SST is an important indicator of ocean conditions and plays a significant role in climate phenomena such as El Niño and La Niña.

- **Sensor:** Thermistor.
- **Placement:** 1 m below the sea surface.
- **Unit:** °C.
- **Typical range:** 24–30 °C.

### 2. Salinity

Measures the concentration of dissolved salts in seawater. Salinity influences water density, ocean circulation, and the mixing of different water masses.

- **Sensor:** Electrical conductivity sensor.
- **Placement:** 1 m below the sea surface.
- **Unit:** Practical Salinity (PSU).
- **Typical range:** 33–36 PSU.

PSU values are not restricted to integers. They can also contain decimal fractions, such as 34.25 or 35.67. Although commonly called a unit, practical salinity is technically dimensionless.

### 3. Ocean Current Speed

Measures the speed of moving seawater. Current measurements help monitor ocean circulation and the transport of heat and nutrients.

- **Sensor:** Acoustic Doppler Current Profiler (ADCP).
- **Placement:** Attached to the underwater mooring, measuring current speed at approximately 10 m depth.
- **Unit:** m/s.
- **Typical range:** 0.0–1.0 m/s.

### 4. Wind Speed

Measures the speed of atmospheric wind above the ocean surface. Wind influences ocean currents, wave formation, and equatorial ocean circulation.

- **Sensor:** Anemometer.
- **Placement:** Approximately 4 m above the sea surface.
- **Unit:** m/s.
- **Typical range:** 0–10 m/s.

### 5. Significant Wave Height

Measures the height of ocean waves. Significant wave height (Hs) represents approximately the average height of the highest one-third of waves.

- **Sensor:** Accelerometer-based wave measurement system.
- **Placement:** Integrated into the floating buoy.
- **Unit:** m.
- **Typical range:** 0.5–2.5 m.

### 6. Wave Period

Measures the average time between successive wave crests passing a fixed point. Wave period helps characterize sea conditions and wave energy.

- **Sensor:** Accelerometer-based wave measurement system.
- **Placement:** Integrated into the floating buoy.
- **Unit:** seconds (s).
- **Typical range:** 5–12 s.

## Monitoring Locations

The five monitoring stations are distributed along the Equator, covering the western, central, and eastern Pacific Ocean.

| Station | Latitude | Longitude | Region |
|---|---|---|---|
| PACIFIC_01 | 0° | 140°E | Western Pacific |
| PACIFIC_02 | 0° | 165°E | Western Pacific |
| PACIFIC_03 | 0° | 170°W | Central Pacific |
| PACIFIC_04 | 0° | 140°W | Central-Eastern Pacific |
| PACIFIC_05 | 0° | 110°W | Eastern Pacific |

## Data Collection

- **Measurement interval:** 1 minute.
- **Number of stations:** 5.
- **Number of monitored parameters:** 6.
- **Data transmission:** Satellite communication.
- **Power supply:** Solar panels and rechargeable batteries.

The measurement ranges presented above are indicative values for normal ocean conditions, not strict physical limits. Actual readings vary depending on geographical location, weather, ocean circulation, and seasonal conditions.

The system is inspired by NOAA's Tropical Atmosphere Ocean (TAO) observing network, which monitors oceanographic and atmospheric conditions across the equatorial Pacific.
