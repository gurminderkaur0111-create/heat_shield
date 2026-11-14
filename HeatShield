import React, { useState, useMemo } from 'react';

// --- TYPES & INTERFACES ---
export interface RegionPreset {
  name: string;
  country: string;
  defaultTemp: number;
  defaultHumidity: number;
  defaultSolar: number;
  defaultConcrete: number;
  defaultCanopy: number;
  currencyOrUnit: string;
}

export interface EnvironmentalMetrics {
  ambientTemp: number;         // °C
  humidity: number;            // %
  greenCanopyPercent: number;  // %
  concreteDensity: number;     // %
  solarRadiation: number;      // W/m²
  windSpeed: number;           // km/h
}

export type PriorityLevel = 'P1 - Emergency Dispatch' | 'P2 - Elevated Alert' | 'P3 - Nominal';

export interface ThermalAssessment {
  wbgtIndex: number;
  heatIndex: number;
  riskLevel: 'Normal (Safe Zone)' | 'Elevated Danger (Caution)' | 'Extreme Heat Hazard (Red Alert)';
  badgeColor: string;
  badgeBg: string;
  maxExhaustionTimeMin: number;
  municipalAction: string;
  dispatchPriority: PriorityLevel;
}

// --- GLOBAL REGION PRESETS ---
const REGION_PRESETS: Record<string, RegionPreset> = {
  INDIA: {
    name: 'Northern Plains (e.g., Punjab / Delhi)',
    country: 'India',
    defaultTemp: 41,
    defaultHumidity: 60,
    defaultSolar: 950,
    defaultConcrete: 85,
    defaultCanopy: 10,
    currencyOrUnit: 'Metric (°C)',
  },
  UAE: {
    name: 'Arabian Peninsula (e.g., Dubai / Abu Dhabi)',
    country: 'United Arab Emirates',
    defaultTemp: 44,
    defaultHumidity: 55,
    defaultSolar: 1050,
    defaultConcrete: 90,
    defaultCanopy: 5,
    currencyOrUnit: 'Metric (°C)',
  },
  USA: {
    name: 'Southwest Desert (e.g., Phoenix, Arizona)',
    country: 'United States',
    defaultTemp: 42,
    defaultHumidity: 25,
    defaultSolar: 1000,
    defaultConcrete: 75,
    defaultCanopy: 15,
    currencyOrUnit: 'Metric (°C)',
  },
  AUS: {
    name: 'Inland Plains (e.g., Western Australia)',
    country: 'Australia',
    defaultTemp: 39,
    defaultHumidity: 30,
    defaultSolar: 980,
    defaultConcrete: 60,
    defaultCanopy: 20,
    currencyOrUnit: 'Metric (°C)',
  },
  BRA: {
    name: 'Urban Metropolis (e.g., São Paulo / Rio)',
    country: 'Brazil',
    defaultTemp: 36,
    defaultHumidity: 75,
    defaultSolar: 850,
    defaultConcrete: 80,
    defaultCanopy: 18,
    currencyOrUnit: 'Metric (°C)',
  },
};

// --- AI LOGIC ENGINE ---
const calculateThermalRisk = (
  metrics: EnvironmentalMetrics,
  targetSector: string,
  countryName: string
): ThermalAssessment => {
  const { ambientTemp, humidity, greenCanopyPercent, concreteDensity, solarRadiation, windSpeed } = metrics;

  // Urban Heat Island (UHI) Microclimate Offset
  const uhiOffset = (concreteDensity * 0.06) - (greenCanopyPercent * 0.05) + (solarRadiation * 0.002) - (windSpeed * 0.2);
  const microclimateTemp = ambientTemp + Math.max(0, uhiOffset);

  // Wet-Bulb Temperature (Tw) Approximation
  const tw = microclimateTemp * Math.atan(0.151977 * Math.pow(humidity + 8.313659, 0.5)) 
    + Math.atan(microclimateTemp + humidity) 
    - Math.atan(humidity - 1.676331) 
    + 0.00391838 * Math.pow(humidity, 1.5) * Math.atan(0.023101 * humidity) - 4.686035;

  // Wet-Bulb Globe Temperature (WBGT) Calculation
  const wbgt = (0.7 * tw) + (0.2 * (microclimateTemp + 2.5)) + (0.1 * microclimateTemp);
  const roundedWbgt = parseFloat(wbgt.toFixed(1));

  // Heat Index Approximation
  const heatIndex = microclimateTemp + (0.5555 * ((6.11 * Math.exp(5417.7530 * ((1 / 273.16) - (1 / (273.15 + tw))))) - 10));
  const roundedHeatIndex = parseFloat(heatIndex.toFixed(1));

  if (roundedWbgt > 32.0) {
    return {
      wbgtIndex: roundedWbgt,
      heatIndex: roundedHeatIndex,
      riskLevel: 'Extreme Heat Hazard (Red Alert)',
      badgeColor: '#dc2626',
      badgeBg: '#fee2e2',
      maxExhaustionTimeMin: 20,
      dispatchPriority: 'P1 - Emergency Dispatch',
      municipalAction: `🚨 CRITICAL EMERGENCY DIRECTIVE [${countryName} / ${targetSector}]: Halt all outdoor labor. Deploy mobile hydration tankers and open municipal cooling centers immediately.`,
    };
  } else if (roundedWbgt > 28.0) {
    return {
      wbgtIndex: roundedWbgt,
      heatIndex: roundedHeatIndex,
      riskLevel: 'Elevated Danger (Caution)',
      badgeColor: '#d97706',
      badgeBg: '#fef3c7',
      maxExhaustionTimeMin: 45,
      dispatchPriority: 'P2 - Elevated Alert',
      municipalAction: `⚠️ PRECAUTIONARY ACTION [${countryName} / ${targetSector}]: Mandate 15-minute shaded rest cycles every 45 minutes. Issue public cellular SMS advisories.`,
    };
  } else {
    return {
      wbgtIndex: roundedWbgt,
      heatIndex: roundedHeatIndex,
      riskLevel: 'Normal (Safe Zone)',
      badgeColor: '#059669',
      badgeBg: '#d1fae5',
      maxExhaustionTimeMin: 120,
      dispatchPriority: 'P3 - Nominal',
      municipalAction: `✅ ROUTINE MONITORING [${countryName} / ${targetSector}]: Thermal indicators remain within safe physiological thresholds.`,
    };
  }
};

// --- MAIN REACT COMPONENT ---
export const HeatShieldDashboard: React.FC = () => {
  const [selectedRegionKey, setSelectedRegionKey] = useState<string>('INDIA');
  const [targetSector, setTargetSector] = useState<string>('Outdoor Construction & Labor');
  
  const currentPreset = REGION_PRESETS[selectedRegionKey];

  const [metrics, setMetrics] = useState<EnvironmentalMetrics>({
    ambientTemp: currentPreset.defaultTemp,
    humidity: currentPreset.defaultHumidity,
    greenCanopyPercent: currentPreset.defaultCanopy,
    concreteDensity: currentPreset.defaultConcrete,
    solarRadiation: currentPreset.defaultSolar,
    windSpeed: 6,
  });

  // Handle global region switch and reset baseline sliders
  const handleRegionChange = (key: string) => {
    setSelectedRegionKey(key);
    const preset = REGION_PRESETS[key];
    setMetrics({
      ambientTemp: preset.defaultTemp,
      humidity: preset.defaultHumidity,
      greenCanopyPercent: preset.defaultCanopy,
      concreteDensity: preset.defaultConcrete,
      solarRadiation: preset.defaultSolar,
      windSpeed: 6,
    });
  };

  const assessment = useMemo(() => {
    return calculateThermalRisk(metrics, targetSector, currentPreset.country);
  }, [metrics, targetSector, currentPreset]);

  const handleInputChange = (field: keyof EnvironmentalMetrics, value: number) => {
    setMetrics((prev) => ({ ...prev, [field]: value }));
  };

  const handleBroadcast = () => {
    alert(
      `🚨 Municipal Alert Dispatched for ${currentPreset.country}!\n\nTarget Sector: ${targetSector}\nWBGT Index: ${assessment.wbgtIndex}°C\nPriority: ${assessment.dispatchPriority}`
    );
  };

  return (
    <div style={styles.container}>
      <header style={styles.header}>
        <div style={styles.headerTitleRow}>
          <h1 style={styles.title}>☀️ HeatShield AI</h1>
          <span style={styles.headerTag}>GLOBAL ENGINE ACTIVE</span>
        </div>
        <p style={styles.subtitle}>Universal Urban Heat Island & Microclimate Early-Warning System</p>
      </header>

      <div style={styles.grid}>
        {/* Input Panel */}
        <div style={styles.column}>
          {/* Global Region Selector Card */}
          <div style={styles.card}>
            <h2 style={styles.cardTitle}>🌍 1. Select Country & Region</h2>
            <select
              value={selectedRegionKey}
              onChange={(e) => handleRegionChange(e.target.value)}
              style={styles.select}
            >
              {Object.entries(REGION_PRESETS).map(([key, preset]) => (
                <option key={key} value={key}>
                  {preset.country} — {preset.name}
                </option>
              ))}
            </select>
          </div>

          <div style={styles.card}>
            <h2 style={styles.cardTitle}>2. Target Vulnerability Sector</h2>
            <select
              value={targetSector}
              onChange={(e) => setTargetSector(e.target.value)}
              style={styles.select}
            >
              <option value="Outdoor Construction & Labor">🏗️ Outdoor Construction & Labor</option>
              <option value="High-Density Residential Slums">🏘️ High-Density Neighborhoods</option>
              <option value="Elderly Care & Medical Hubs">🏥 Elderly Care & Medical Hubs</option>
              <option value="Educational & School Playgrounds">🏫 Educational & Playgrounds</option>
            </select>
          </div>

          <div style={styles.card}>
            <h2 style={styles.cardTitle}>3. Microclimate Telemetry Controls</h2>

            <SliderControl
              label="🌡️ Ambient Air Temp"
              value={metrics.ambientTemp}
              unit="°C"
              min={20}
              max={52}
              onChange={(val) => handleInputChange('ambientTemp', val)}
            />

            <SliderControl
              label="💧 Relative Humidity"
              value={metrics.humidity}
              unit="%"
              min={10}
              max={95}
              onChange={(val) => handleInputChange('humidity', val)}
            />

            <SliderControl
              label="☀️ Solar Radiation"
              value={metrics.solarRadiation}
              unit="W/m²"
              min={200}
              max={1200}
              step={25}
              onChange={(val) => handleInputChange('solarRadiation', val)}
            />

            <SliderControl
              label="🏢 Concrete Density"
              value={metrics.concreteDensity}
              unit="%"
              min={10}
              max={95}
              onChange={(val) => handleInputChange('concreteDensity', val)}
            />

            <SliderControl
              label="🌳 Greenery Canopy Cover"
              value={metrics.greenCanopyPercent}
              unit="%"
              min={2}
              max={60}
              onChange={(val) => handleInputChange('greenCanopyPercent', val)}
            />
          </div>
        </div>

        {/* Analytics & Output Panel */}
        <div style={styles.column}>
          <div style={styles.card}>
            <h2 style={styles.cardTitle}>4. AI Analytics & Municipal Dispatch</h2>

            <div style={styles.metricRow}>
              <MetricCard label="WBGT Index" value={`${assessment.wbgtIndex}°C`} />
              <MetricCard label="Apparent Heat" value={`${assessment.heatIndex}°C`} />
              <MetricCard
                label="Max Exposure"
                value={`${assessment.maxExhaustionTimeMin} min`}
                highlightColor={assessment.badgeColor}
              />
            </div>

            <div
              style={{
                ...styles.statusBanner,
                backgroundColor: assessment.badgeBg,
                color: assessment.badgeColor,
                borderColor: assessment.badgeColor,
              }}
            >
              <div style={{ fontWeight: '800', fontSize: '1.2rem' }}>{assessment.riskLevel}</div>
              <div style={{ fontSize: '0.85rem', marginTop: '4px', opacity: 0.9 }}>
                Region: {currentPreset.country} | Priority: {assessment.dispatchPriority}
              </div>
            </div>

            <div style={styles.actionBox}>
              <h4 style={styles.actionHeader}>Automated Municipal Directive</h4>
              <p style={styles.actionText}>{assessment.municipalAction}</p>
            </div>

            <button
              onClick={handleBroadcast}
              style={{ ...styles.button, backgroundColor: assessment.badgeColor }}
            >
              Broadcast Municipal Advisory ({currentPreset.country})
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};

// --- SUB-COMPONENTS & STYLES ---
interface SliderProps {
  label: string;
  value: number;
  unit: string;
  min: number;
  max: number;
  step?: number;
  onChange: (val: number) => void;
}

const SliderControl: React.FC<SliderProps> = ({ label, value, unit, min, max, step = 1, onChange }) => (
  <div style={styles.fieldGroup}>
    <div style={styles.labelRow}>
      <label style={styles.label}>{label}</label>
      <span style={styles.valBadge}>{value} {unit}</span>
    </div>
    <input
      type="range"
      min={min}
      max={max}
      step={step}
      value={value}
      onChange={(e) => onChange(parseFloat(e.target.value))}
      style={styles.slider}
    />
  </div>
);

interface MetricCardProps {
  label: string;
  value: string;
  highlightColor?: string;
}

const MetricCard: React.FC<MetricCardProps> = ({ label, value, highlightColor }) => (
  <div style={styles.metricBox}>
    <span style={{ ...styles.metricValue, color: highlightColor || '#0f172a' }}>{value}</span>
    <span style={styles.metricLabel}>{label}</span>
  </div>
);

const styles: { [key: string]: React.CSSProperties } = {
  container: {
    maxWidth: '1080px',
    margin: '0 auto',
    padding: '30px 20px',
    fontFamily: 'Inter, system-ui, -apple-system, sans-serif',
    color: '#1f2937',
    backgroundColor: '#f8fafc',
    minHeight: '100vh',
  },
  header: { marginBottom: '28px' },
  headerTitleRow: { display: 'flex', alignItems: 'center', gap: '12px' },
  title: { fontSize: '2.4rem', fontWeight: '900', color: '#ea580c', margin: 0 },
  headerTag: {
    backgroundColor: '#ffedd5',
    color: '#c2410c',
    fontSize: '0.75rem',
    fontWeight: '800',
    padding: '4px 10px',
    borderRadius: '20px',
    border: '1px solid #fdba74',
  },
  subtitle: { fontSize: '1rem', color: '#64748b', marginTop: '6px' },
  grid: { display: 'grid', gridTemplateColumns: 'repeat(auto-fit, minmax(340px, 1fr))', gap: '24px' },
  column: { display: 'flex', flexDirection: 'column', gap: '24px' },
  card: {
    backgroundColor: '#ffffff',
    borderRadius: '16px',
    padding: '24px',
    boxShadow: '0 4px 20px -2px rgba(0, 0, 0, 0.05)',
    border: '1px solid #e2e8f0',
  },
  cardTitle: { fontSize: '1.15rem', fontWeight: '700', color: '#0f172a', marginTop: 0, marginBottom: '18px' },
  fieldGroup: { marginBottom: '16px' },
  labelRow: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '6px' },
  label: { fontSize: '0.85rem', fontWeight: '600', color: '#334155' },
  valBadge: { fontSize: '0.8rem', fontWeight: '700', backgroundColor: '#ffedd5', color: '#c2410c', padding: '3px 8px', borderRadius: '6px' },
  slider: { width: '100%', accentColor: '#ea580c', cursor: 'pointer' },
  select: {
    width: '100%',
    padding: '12px',
    borderRadius: '10px',
    border: '1px solid #cbd5e1',
    fontSize: '0.95rem',
    backgroundColor: '#f8fafc',
    color: '#0f172a',
    fontWeight: '600',
  },
  metricRow: { display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '12px', marginBottom: '20px' },
  metricBox: { backgroundColor: '#f1f5f9', borderRadius: '12px', padding: '14px 10px', textAlign: 'center', border: '1px solid #e2e8f0' },
  metricValue: { display: 'block', fontSize: '1.4rem', fontWeight: '900' },
  metricLabel: { fontSize: '0.7rem', color: '#64748b', fontWeight: '700', textTransform: 'uppercase', marginTop: '4px' },
  statusBanner: { padding: '16px', borderRadius: '12px', textAlign: 'center', border: '2px solid', marginBottom: '20px' },
  actionBox: { backgroundColor: '#f8fafc', borderLeft: '4px solid #ea580c', padding: '16px', borderRadius: '8px', marginBottom: '20px' },
  actionHeader: { margin: '0 0 6px 0', fontSize: '0.9rem', color: '#0f172a' },
  actionText: { margin: 0, fontSize: '0.85rem', color: '#475569', lineHeight: '1.5' },
  button: { width: '100%', padding: '14px', color: '#ffffff', fontWeight: '800', fontSize: '0.95rem', border: 'none', borderRadius: '10px', cursor: 'pointer' },
};

export default HeatShieldDashboard;
