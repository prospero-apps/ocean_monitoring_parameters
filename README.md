# Ocean Monitoring Parameters

## Overview

The simulated ocean monitoring system consists of five autonomous monitoring buoys deployed along the Equator in the Pacific Ocean. Each buoy is equipped with sensors to measure six oceanographic and meteorological parameters.

The system monitors normal ocean conditions, collecting measurements every **1 minute**. The selected parameters provide information about ocean temperature, salinity, circulation, wind, and wave conditions.

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
- **Typical range:** 0.05–1.0 m/s.

### 4. Wind Speed

Measures the speed of atmospheric wind above the ocean surface. Wind influences ocean currents, wave formation, and equatorial ocean circulation.

- **Sensor:** Anemometer.
- **Placement:** Approximately 4 m above the sea surface.
- **Unit:** m/s.
- **Typical range:** 2–10 m/s.

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
