import React, { useMemo, useRef } from 'react';
import { useFrame, useThree } from '@react-three/fiber';
import { ScrollControls, useScroll, Text, Float, Line, Sparkles } from '@react-three/drei';
import * as THREE from 'three';

// --- CONFIGURATION ---
const LINE_POINTS = [
  new THREE.Vector3(0, 2, -5),   // Start (Just inside door)
  new THREE.Vector3(0, 2, -12),  // Rose Day
  new THREE.Vector3(-4, 2, -20), // Propose Day
  new THREE.Vector3(3, 2, -30),  // Chocolate Day
  new THREE.Vector3(-3, 2.5, -40), // Teddy Day
  new THREE.Vector3(2, 3, -50),  // Promise Day
  new THREE.Vector3(0, 3, -60),  // Hug Day
  new THREE.Vector3(-2, 3, -70), // Kiss Day
  new THREE.Vector3(0, 3.5, -80) // Valentine's Day
];

// Create the smooth curve
const CURVE = new THREE.CatmullRomCurve3(LINE_POINTS, false, 'catmullrom', 0.5);

// --- COMPONENT: Camera Handler (Moves camera on scroll) ---
function ScrollHandler() {
  const scroll = useScroll();
  const { camera } = useThree();
  
  useFrame(() => {
    // 1. Get point on curve based on scroll offset (0 to 1)
    // We limit the range slightly so we don't fall off the end
    const point = CURVE.getPointAt(scroll.offset * 0.95);
    
    // 2. Get a point slightly ahead to look at (tangent)
    const lookAtPoint = CURVE.getPointAt(Math.min(scroll.offset * 0.95 + 0.05, 1));

    // 3. Move camera
    camera.position.copy(point);
    camera.lookAt(lookAtPoint);
  });
  return null;
}

// --- COMPONENT: Visual Path (The glowing line) ---
function PathLine() {
  const points = useMemo(() => CURVE.getPoints(100), []);
  return (
    <group>
      {/* The visible glowing line */}
      <Line 
        points={points} 
        color="#ff0055" 
        lineWidth={3} 
        opacity={0.4} 
        transparent 
      />
      {/* Sparkles following the path */}
      {points.map((p, i) => i % 5 === 0 && (
         <Sparkles key={i} position={p} count={5} scale={3} size={4} color="gold" />
      ))}
    </group>
  );
}

// --- COMPONENT: Day Checkpoint ---
function Checkpoint({ position, label, idx }) {
    return (
        <Float speed={2} rotationIntensity={0.2} floatIntensity={0.5}>
            <group position={position}>
                {/* The Label */}
                <Text 
                    position={[0, 1.5, 0]} 
                    fontSize={0.8} 
                    color="#ffd700" 
                    anchorX="center" 
                    anchorY="middle"
                    font="https://fonts.gstatic.com/s/raleway/v14/1Ptrg8zYS_SKggPNwK4vaqI.woff"
                    toneMapped={false}
                >
                    {label}
                </Text>
                
                {/* A glowing marker orb */}
                <mesh position={[0, 0, 0]}>
                    <sphereGeometry args={[0.3, 16, 16]} />
                    <meshStandardMaterial color="#ff0055" emissive="#ff0055" emissiveIntensity={2} />
                </mesh>

                {/* Date Label */}
                <Text 
                    position={[0, -0.6, 0]} 
                    fontSize={0.4} 
                    color="#ffffff" 
                    anchorX="center" 
                    toneMapped={false}
                >
                    Feb {7 + idx}
                </Text>
            </group>
        </Float>
    );
}

// --- MAIN HUB WORLD COMPONENT ---
export default function HubWorld() {
  const days = [
    "Start Journey", "Rose Day", "Propose Day", "Chocolate Day", 
    "Teddy Day", "Promise Day", "Hug Day", "Kiss Day", "Valentine's Day"
  ];

  return (
    // Pages = roughly how long the scroll is. 
    // Damping = smooth scrolling friction
    <ScrollControls pages={8} damping={0.3}>
      <ScrollHandler />
      
      <PathLine />

      {/* Place Checkpoints along the curve */}
      {LINE_POINTS.map((pos, i) => (
          // We offset the text slightly to the side so it doesn't block the camera
          <Checkpoint 
            key={i} 
            position={[pos.x + 1.5, pos.y, pos.z]} 
            label={days[i]} 
            idx={i-1} // i-1 because first point is "Start"
          />
      ))}
    </ScrollControls>
  );
}
