#if anything went wrong 
import React, { useState, useCallback, forwardRef, useEffect } from "react";
import { useDroppable } from "@dnd-kit/core";
import { SortableContext, rectSortingStrategy } from "@dnd-kit/sortable";
import { Badge } from "@/components/ui/badge";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Trash2, X, Zap, ChevronUp, ChevronDown, RotateCcw, Power } from "lucide-react";
import { Icon, ModuleSize } from "@/types";
import SortableIconSlot from "./SortableIconSlot";
import "./Panel.css";
import o from '@/assest/curtains.png'
import p from '@/assest/smart-curtain.png'
import y from '@/assest/curtain.png'
import z from '@/assest/shower-curtains.png'
import x from '@/assest/wall-1.jpg'
import wall1 from '@/assest/wall-1.jpg';
import wall2 from '@/assest/wall-2.jpeg';
import wall3 from '@/assest/wall-3.jpeg';
import wall4 from '@/assest/walll-3.jpeg';
import wall5 from '@/assest/wall-5.jpeg';
import wall6 from '@/assest/walll-2.jpeg';
import wall7 from '@/assest/walll-1.jpeg';
import blue from '@/assest/lblue.jpeg'
// Extended Icon interface for panel instances
interface PanelIcon extends Icon {
  instanceId: string;
}

interface TouchPanelPreviewProps {
  size: ModuleSize;
  variant: string;
  glass: string;
  frame: string;
  icons: (PanelIcon | null)[]; // Changed to slot-based array
  onRemoveIcon: (instanceId: string) => void;
  onIconsChange?: (icons: (PanelIcon | null)[]) => void;
  onPlaceIcon?: (icon: Icon, slotIndex: number) => void; // New prop for placing icons
  onMoveIcon?: (fromSlot: number, toSlot: number) => void; // New prop for moving icons
}

const TouchPanelPreview = forwardRef<HTMLDivElement, TouchPanelPreviewProps>(({
  size,
  variant,
  glass,
  frame,
  icons: initialIcons,
  onRemoveIcon,
  onIconsChange,
  onPlaceIcon,
  onMoveIcon,
}, ref) => {
  const [icons, setIcons] = useState<(PanelIcon | null)[]>([]);
  const [lastPanelConfig, setLastPanelConfig] = useState<string>('');
  const wallOptions = [blue,wall1, wall2, wall3, wall4, wall6, wall7];
  const [selectedWall, setSelectedWall] = useState(wall1);
  // Create a unique key for the current panel configuration
  const getCurrentPanelConfig = useCallback(() => {
    return `${size}-${variant}-${glass}-${frame}`;
  }, [size, variant, glass, frame]);

  // Reset icons when panel configuration changes
  useEffect(() => {
    const currentConfig = getCurrentPanelConfig();
    
    if (lastPanelConfig && lastPanelConfig !== currentConfig) {
      // Panel configuration changed - clear all icons
      const layoutConfig = getLayoutConfig();
      const totalSlots = layoutConfig.switchSlots + 
                        (layoutConfig.curtainSlots || 0) + 
                        (layoutConfig.fanSlots || 0) + 
                        (layoutConfig.plugSlots || 0);
      
      const emptyIcons = new Array(totalSlots).fill(null);
      setIcons(emptyIcons);
      
      // Notify parent component about the reset
      if (onIconsChange) {
        onIconsChange(emptyIcons);
      }
      
      // Remove all existing icons from parent state
      initialIcons.forEach((icon) => {
        if (icon) {
          onRemoveIcon(icon.instanceId);
        }
      });
    } else if (!lastPanelConfig) {
      // First load - use initial icons
      setIcons(initialIcons);
    }
    
    setLastPanelConfig(currentConfig);
  }, [size, variant, glass, frame, getCurrentPanelConfig, lastPanelConfig, onIconsChange, onRemoveIcon]);

  // Update icons when initialIcons changes (but only if panel config hasn't changed)
  useEffect(() => {
    const currentConfig = getCurrentPanelConfig();
    if (lastPanelConfig === currentConfig) {
      setIcons(initialIcons);
    }
  }, [initialIcons, lastPanelConfig, getCurrentPanelConfig]);

  const handleDeleteIcon = useCallback(
    (instanceId: string) => {
      const updatedIcons = icons.map(icon => 
        icon?.instanceId === instanceId ? null : icon
      );
      setIcons(updatedIcons);
      if (onIconsChange) onIconsChange(updatedIcons);
      onRemoveIcon(instanceId);
    },
    [icons, onRemoveIcon, onIconsChange]
  );

  const handleClearAllIcons = useCallback(() => {
    const clearedIcons = new Array(icons.length).fill(null);
    setIcons(clearedIcons);
    if (onIconsChange) onIconsChange(clearedIcons);
    
    // Remove all non-null icons
    icons.forEach((icon) => {
      if (icon) {
        onRemoveIcon(icon.instanceId);
      }
    });
  }, [icons, onRemoveIcon, onIconsChange]);

  // Get layout configuration based on size and variant
  const getLayoutConfig = () => {
    const hasPlug = variant.toLowerCase().includes("plug");
    const hasFans = variant.toLowerCase().includes("fan");
    const hasCurtains = variant.toLowerCase().includes("curtains");
    const theme = variant.toLowerCase().includes("theme")
    
    switch (size) {
      case "2":
        if (variant === "4 switches") {
          // 2x2 grid with 4 interactive switch slots
          return {
            cols: "grid-cols-2",
            rows: "grid-rows-2",
            panelSize: "w-[350px] h-[350px]",
            slotSize: "w-[60px] h-[60px]",
            gap: "gap-x-5 gap-y-10",
            fanGap: "gap-x-5 gap-y-2", // Consistent with main grid x-gap
            padding: "p-4",
            switchSlots: 4, // 4 interactive switch slots
            curtainSlots: 0,
            plugSlots: 0,
            fanSlots: 0,
            hasPlug: false,
            hasFans: false,
            hasCurtains: false,
            // No hardcoded icons for interactive switch module
          };
        } else if (variant === "2*2 curtains") {
          // 2x2 grid with 4 hardcoded curtain icons
          return {
            cols: "grid-cols-2",
            rows: "grid-rows-2", 
            panelSize: "w-[350px] h-[350px]",
            slotSize: "w-[60px] h-[60px]",
            curtainSize: "w-[60px] h-[60px]",
            gap: "gap-x-5 gap-y-10",
            fanGap: "gap-x-5 gap-y-2",
            padding: "p-4",
            switchSlots: 0, // No interactive switch slots
            plugSlots: 0,
            fanSlots: 0,
          
            hasPlug: false,
            hasFans: false,
            hasCurtains: true,
            // 2x2 curtain layout - 4 hardcoded curtain icons
            slotIcons: {
              0: o,  // Top-left curtain
              1: p,  // Top-right curtain
              2: y,  // Bottom-left curtain
              3: z   // Bottom-right curtain
            },
            curtainSlots: 4, // 4 curtain positions
            isFullCurtainMode: true // Flag to indicate this is pure curtain mode
          };
        } else if (variant === "4 Switches + 2 curtains") {
          // Legacy variant - keeping for backward compatibility
          return {
            cols: "grid-cols-3",
            rows: "grid-rows-2", 
            panelSize: "w-[350px] h-[350px]",
            slotSize: "w-[60px] h-[60px]",
            curtainSize: "w-[60px] h-[60px]",
            gap: "gap-x-4 gap-y-8",
            fanGap: "gap-x-4 gap-y-2",
            padding: "p-6",
            switchSlots: 4,
            plugSlots: 0,
            fanSlots: 0,
            hasCurtains: true,
          };
        }
        // Default fallback for size "2"
        return {
            cols: "grid-cols-2",
            rows: "grid-rows-2", 
            panelSize: "w-[350px] h-[350px]",
            slotSize: "w-[60px] h-[60px]",
            curtainSize: "w-[60px] h-[60px]",
            gap: "gap-x-5 gap-y-10",
            fanGap: "gap-x-5 gap-y-2",
            padding: "p-4",
            switchSlots: 0, // No interactive switch slots
            plugSlots: 0,
            fanSlots: 0,
          
            hasPlug: false,
            hasFans: false,
            hasCurtains: true,
            // 2x2 curtain layout - 4 hardcoded curtain icons
            slotIcons: {
              0: o,  // Top-left curtain
              1: p,  // Top-right curtain
              2: z,  // Bottom-left curtain
              3: y   // Bottom-right curtain
            },
            curtainSlots: 4, // 4 curtain positions
            isFullCurtainMode: true // Flag to indicate this is pure curtain mode
          };

      case "4":
        if (variant === "8 switches") {
          return {
            cols: "grid-cols-4",
            rows: "grid-rows-2",
            panelSize: "w-[520px] h-[320px]",
            slotSize: "w-[60px] h-[60px]",
            gap: "gap-x-4 gap-y-12",
            fanGap: "gap-x-4 gap-y-3",
            padding: "p-8",
            switchSlots: 8,
            plugSlots: 0,
            fanSlots: 0,
          };
        } else if (variant === "4 switches + 1 plug") {
          return {
            cols: "grid-cols-2",
            rows: "grid-rows-2",
            panelSize: "w-[530px] h-[320px]",
            slotSize: "w-[60px] h-[60px]",
            plugSize: "w-[170px] h-[170px]",
            gap: "gap-x-5 gap-y-12",
            fanGap: "gap-x-5 gap-y-12",
            padding: "p-8",
            switchSlots: 4,
            plugSlots: 1,
            fanSlots: 0,
            hasPlug: true,
          };
        } else if (variant === "4 switches + fan") {
          return {
            cols: "grid-cols-2",
            rows: "grid-rows-2",
            panelSize: "w-[530px] h-[320px]",
            slotSize: "w-[55px] h-[55px]",
            fanSize: "w-[65x] h-[65px]",
            gap: "gap-x-5 gap-y-12",
            fanGap: "gap-x-5 gap-y-12",
            padding: "p-8",
            switchSlots: 4,
            plugSlots: 0,
            fanSlots: 4, // Fan has 4 slots: fan+, fan-, fan, power
            hasFans: true,
          };
        }
        

        return {
          cols: "grid-cols-2",
          rows: "grid-rows-2",
          panelSize: "w-[530px] h-[320px]",
          slotSize: "w-[58px] h-[58px]",
          plugSize: "w-[160px] h-[160px] border-none  bg-transparent  shadow-none",
          gap: "gap-x-5 gap-y-12",
          fanGap: "gap-x-5 gap-y-12",
          padding: "p-8",
          switchSlots: 4,
          plugSlots: 1,
          fanSlots: 0,
          hasPlug:true
        };

      case "6":
        if (variant === "10 Switch") {
          return {
            cols: "grid-cols-5",
            rows: "grid-rows-2",
            panelSize: "w-[720px] h-[320px]",
            slotSize: "w-[60px] h-[60px]",
            gap: "gap-x-4 gap-y-12",
            fanGap: "gap-x-4 gap-y-3",
            padding: "p-8",
            switchSlots: 10,
            plugSlots: 0,
            fanSlots: 0,
          };
        } else if (variant === "8 switch + 1 Plug") {
          return {
            cols: "grid-cols-4",
            rows: "grid-rows-2",
            panelSize: "w-[720px] h-[320px]",
            slotSize: "w-[60px] h-[60px]",
            plugSize: "w-[170px] h-[170px] border-none  bg-transparent  shadow-none",
            gap: "gap-x-4 gap-y-12",
            fanGap: "gap-x-4 gap-y-3",
            padding: "p-8",
            switchSlots: 8,
            plugSlots: 1,
            fanSlots: 0,
            hasPlug: true,
          };
        } else if (variant === "4 switches + 1 fan + 1 Plug") {
          return {
            cols: "grid-cols-2",
            rows: "grid-rows-2",
            panelSize: "w-[720px] h-[320px]",
            slotSize: "w-[55px] h-[55px]",
            plugSize: "w-[157px] h-[155px]  bg-transparent border-none shadow-none",
            fanSize: "w-[60px] h-[60px] ",
            gap: "gap-x-5 gap-y-12",
            fanGap: "gap-x-5 gap-y-12",
            padding: "p-8",
            switchSlots: 4,
            plugSlots: 1,
            fanSlots: 4, // Fan has 4 slots: fan+, fan-, fan, power
            hasPlug: true,
            hasFans: true,
          };
        }
        return {
          cols: "grid-cols-5",
          rows: "grid-rows-2",
          panelSize: "w-[720px] h-[320px]",
          slotSize: "w-[60px] h-[60px]",
          gap: "gap-x-5 gap-y-12",
          fanGap: "gap-x-5 gap-y-12",
          padding: "p-10",
          switchSlots: 10,
          plugSlots: 0,
          fanSlots: 0,
        };

      case "8":
        if (variant === "10 switches + 1 Plug") {
          return {
            cols: "grid-cols-5",
            rows: "grid-rows-2",
            panelSize: "w-[760px] h-[300px]",
            slotSize: "w-[60px] h-[60px]",
            plugSize: "w-[170px] h-[170px] border-none  bg-transparent  shadow-none",
            gap: "gap-x-4 gap-y-12",
            fanGap: "gap-x-4 gap-y-3",
            padding: "p-12",
            switchSlots: 10,
            plugSlots: 1,
            fanSlots: 0,
            hasPlug: true,
            hasFans:false
          };
        }
        return {
          cols: "grid-cols-3",
          rows: "grid-rows-2",
          panelSize: "w-[760px] h-[300px]",
          slotSize: "w-[55px] h-[55px]",
          plugSize: "w-[155px] h-[155px] border-none bg-white shadow-none",
          fanSize: "w-[66px] h-[66px]",
          gap: "gap-x-5 gap-y-12",
          fanGap: "gap-x-5 gap-y-12",
          padding: "p-12",
          switchSlots: 6,
          plugSlots: 1,
          fanSlots: 4,
          hasFans:true,
          hasPlug:true
        };

      default:
        return {
          cols: "grid-cols-2",
          rows: "grid-rows-2",
          panelSize: "w-[400px] h-[320px]",
          slotSize: "w-[80px] h-[80px]",
          gap: "gap-x-4 gap-y-10",
          fanGap: "gap-x-4 gap-y-2",
          padding: "p-8",
          switchSlots: 4,
          plugSlots: 0,
          fanSlots: 0,
          curtainSlots: 0,
          hasPlug: false,
          hasFans: false,
          hasCurtains: false,
        };
    }
  };

  const getGlassEffect = () => {
    switch (glass) {
      case "Black":
        return {
          background: "linear-gradient(135deg, #1e1e1e 0%, #0d0d0d 100%)",
          overlay: "linear-gradient(150deg, rgba(255,255,255,0.05) 20%, transparent 70%)",
        };
      case "White":
        return {
         
  background: "linear-gradient(135deg, #ffffff 0%, #ffffff 100%)",
  overlay: "linear-gradient(150deg, rgba(0,0,0,0) 0%, transparent 100%)",


        };
      default: // Default to Black if no valid option
        return {
          background: "linear-gradient(135deg, #1e1e1e 0%, #0d0d0d 100%)",
          overlay: "linear-gradient(150deg, rgba(255,255,255,0.05) 20%, transparent 70%)",
        };
    }
  };

  const getFrameStyle = () => {
    switch (frame) {
      case "Gold":
        return "border-amber-400 shadow-amber-400/20";
      case "Silver":
        return "border-gray-300 shadow-gray-300/20";
      case "White":
        return "border-white shadow-white/20";
      default: // Black
        return "border-gray-700 shadow-gray-700/20";
    }
  };

  const layoutConfig = getLayoutConfig();
  const glassEffect = getGlassEffect();
  const frameStyle = getFrameStyle();

  // Create individual droppable zones for each slot
  const SlotDropZone: React.FC<{ slotIndex: number; children: React.ReactNode }> = ({ slotIndex, children }) => {
    const { setNodeRef, isOver } = useDroppable({
      id: `slot-${slotIndex}`,
      data: { type: "slot", slotIndex }
    });

    return (
      <div
        ref={setNodeRef}
        className={`
          transition-all duration-200
          ${isOver ? "ring-2 ring-blue-400 ring-opacity-50 bg-blue-400/10" : ""}
        `}
      >
        {children}
      </div>
    );
  };

  // Render curtain section
  const renderCurtainSection = () => {
    if (!layoutConfig.hasCurtains) return null;

    const curtainStartIndex = layoutConfig.switchSlots;
    
    return (
      <div className="flex flex-col items-center justify-center ml-6">
        <div className="flex flex-col space-y-3">
          {Array.from({ length: layoutConfig.curtainSlots || 0 }, (_, index) => {
            const slotIndex = curtainStartIndex + index;
            const icon = icons[slotIndex];
            
            return (
              <SlotDropZone key={`curtain-${index}`} slotIndex={slotIndex}>
                <div
                  className={`
                    relative flex flex-col items-center justify-between 
                    rounded-xl ${layoutConfig.curtainSize} group
                    transition-all duration-100
                    ${
                      icon
                        ? glass === "White" 
                          ? "bg-black/10 border-2 border-black/20 shadow-lg"
                          : "bg-white/10 border-2 border-white/20 shadow-lg"
                        : glass === "White"
                          ? "border-2 border-black/10 border-dashed hover:border-black/30"
                          : "border-2 border-white/10 border-dashed hover:border-white/30"
                    }
                  `}
                >
                  {icon ? (
                    <>
                      <SortableIconSlot
                        id={icon.instanceId}
                        icon={icon}
                        slotIndex={slotIndex}
                        onRemove={handleDeleteIcon}
                      />
                      <button
                        onClick={() => handleDeleteIcon(icon.instanceId)}
                        className="absolute -top-2 -right-2 w-6 h-6 bg-red-500 hover:bg-red-600 text-white rounded-full flex items-center justify-center shadow-lg opacity-0 group-hover:opacity-100 transition-all duration-200 z-20 border-2 border-white"
                        aria-label={`Delete ${icon.name || "curtain icon"}`}
                      >
                        <X className="w-3 h-3" />
                      </button>
                    </>
                  ) : (
                    <div className={`flex flex-col items-center justify-center h-full py-4 ${glass === "White" ? "text-black/50" : "text-white/50"}`}>
                      {/* Curtain icon representation */}
                      <div className="flex flex-col items-center">
                        <div className={`w-8 h-1 rounded-full mb-2 ${glass === "White" ? "bg-black/40" : "bg-white/40"}`}></div>
                        <div className="flex space-x-0.5">
                          {Array.from({ length: 5 }, (_, i) => (
                            <div key={i} className={`w-1.5 h-16 rounded-sm ${glass === "White" ? "bg-black/40" : "bg-white/40"}`}></div>
                          ))}
                        </div>
                        <div className="text-[10px] font-medium mt-2 text-center">
                        <br/>{index + 1}
                        </div>
                      </div>
                    </div>
                  )}
                </div>
              </SlotDropZone>
            );
          })}
        </div>
      </div>
    );
  };

  // Render fan section with 4 control slots
  const renderFanSection = () => {
    if (!layoutConfig.hasFans) return null;

    const fanStartIndex = layoutConfig.switchSlots + (layoutConfig.curtainSlots || 0);
    const fanControls = [
      { 
        icon: "speed-up", 
        label: "SPEED+", 
        name: "fan-speed-up",
        customIcon: (
          <div className="flex flex-col items-center">
            <div className="w-4 h-4 relative">
              <div className="absolute inset-0 border-2 border-current rounded-full"></div>
              <div className="absolute top-1 left-1 right-1 bottom-1 flex items-center justify-center">
                <ChevronUp className="w-2 h-2" />
              </div>
            </div>
          </div>
        )
      },
      { 
        icon: "speed-down", 
        label: "SPEED-", 
        name: "fan-speed-down",
        customIcon: (
          <div className="flex flex-col items-center">
            <div className="w-4 h-4 relative">
              <div className="absolute inset-0 border-2 border-current rounded-full"></div>
              <div className="absolute top-1 left-1 right-1 bottom-1 flex items-center justify-center">
                <ChevronDown className="w-2 h-2" />
              </div>
            </div>
          </div>
        )
      },
      { 
        icon: "oscillate", 
        label: "OSC", 
        name: "fan-oscillate",
        customIcon: (
          <div className="flex flex-col items-center">
            <div className="w-4 h-4 relative">
              <div className="absolute inset-0 border-2 border-current rounded-full"></div>
              <div className="absolute top-1 left-1 right-1 bottom-1 flex items-center justify-center">
                <RotateCcw className="w-2 h-2" />
              </div>
            </div>
          </div>
        )
      },
      { 
        icon: "power", 
        label: "POWER", 
        name: "fan-power",
        customIcon: (
          <div className="flex flex-col items-center">
            <div className="w-4 h-4 relative">
              <div className="absolute inset-0 border-2 border-current rounded-full"></div>
              <div className="absolute top-1 left-1 right-1 bottom-1 flex items-center justify-center">
                <Power className="w-2 h-2" />
              </div>
            </div>
          </div>
        )
      }
    ];

    return (
      <div className="flex flex-col items-center  justify-center ml-6">
        <div className={`grid grid-cols-2 ${layoutConfig.fanGap}`}>
          {fanControls.map((control, index) => {
            const slotIndex = fanStartIndex + index;
            const icon = icons[slotIndex];
            
            return (
              <SlotDropZone key={`fan-${index}`} slotIndex={slotIndex}>
                <div
                  className={`
                    relative flex flex-col items-center justify-center 
                    rounded-xl w-[55px] h-[55px] group
                    transition-all duration-100
                    ${
                      icon
                        ? glass === "White"
                          ? "bg-black/10 border-2 border-black/20 shadow-lg"
                          : "bg-white/10 border-2 border-white/20 shadow-lg"
                        : glass === "White"
                          ? "border-2 border-black/10 border-dashed hover:border-black/30"
                          : "border-2 border-white/10 border-dashed hover:border-white/30"
                    }
                  `}
                >
                  {icon ? (
                    <>
                      <SortableIconSlot
                        id={icon.instanceId}
                        icon={icon}
                        slotIndex={slotIndex}
                        onRemove={handleDeleteIcon}
                      />
                      <button
                        onClick={() => handleDeleteIcon(icon.instanceId)}
                        className="absolute -top-2 -right-2 w-6 h-6 bg-red-500 hover:bg-red-600 text-white rounded-full flex items-center justify-center shadow-lg opacity-0 group-hover:opacity-100 transition-all duration-200 z-20 border-2 border-white"
                        aria-label={`Delete ${icon.name || "fan control"}`}
                      >
                        <X className="w-3 h-3" />
                      </button>
                    </>
                  ) : (
                    <div className={`flex flex-col items-center justify-center ${glass === "White" ? "text-black/50" : "text-white/50"}`}>
                      <div className="mb-1">
                        {control.customIcon}
                      </div>
                      <span className="text-[8px] font-medium text-center leading-tight">{control.label}</span>
                    </div>
                  )}
                </div>
              </SlotDropZone>
            );
          })}
        </div>
        
        {/* Fan label */}
        {/* <div className="mt-2 text-white/40 text-[10px] font-medium text-center">
          FAN CONTROLS
        </div> */}
      </div>
    );
  };

  // Render plug section
  const renderPlugSection = () => {
    if (!layoutConfig.hasPlug) return null;

    const plugStartIndex = layoutConfig.switchSlots + (layoutConfig.curtainSlots || 0) + (layoutConfig.fanSlots || 0);
    const icon = icons[plugStartIndex];

    return (
      <div className="flex flex-col items-center justify-center ml-6">
        <SlotDropZone slotIndex={plugStartIndex}>
          <div
            className={`
              relative flex items-center justify-center 
              rounded-xl ${layoutConfig.plugSize} group
              transition-all duration-100
              ${
                icon
                  ? glass === "White"
                    ? "bg-black/10 border-2 border-black/20 shadow-lg"
                    : "bg-white/10 border-2 border-white/20 shadow-lg"
                  : glass === "White"
                    ? "bg-black/15 border-2 border-black/30 shadow-lg hover:bg-black/20 hover:border-black/40"
                    : "bg-white/15 border-2 border-white/30 shadow-lg hover:bg-white/20 hover:border-white/40"
              }
            `}
          >
            {icon ? (
              <>
                <SortableIconSlot
                  id={icon.instanceId}
                  icon={icon}
                  slotIndex={plugStartIndex}
                  onRemove={handleDeleteIcon}
                />
                <button
                  onClick={() => handleDeleteIcon(icon.instanceId)}
                  className="absolute -top-2 -right-2 w-6 h-6 bg-red-500 hover:bg-red-600 text-white rounded-full flex items-center justify-center shadow-lg opacity-0 group-hover:opacity-100 transition-all duration-200 z-20 border-2 border-white"
                  aria-label={`Delete ${icon.name || "plug"}`}
                >
                  <X className="w-3 h-3" />
                </button>
              </>
            ) : (
              <div className="w-27 h-27  flex items-center justify-center">
                <svg
                  viewBox="0 0 140 120"
                  className="w-full h-full"
                  style={{ filter: glass === "White" ? "brightness(0)" : "brightness(0) invert(1)" }}
                >
                  <circle cx="70" cy="30" r="15" fill="currentColor" />
                  <circle cx="35" cy="95" r="10" fill="currentColor" />
                  <circle cx="105" cy="95" r="10" fill="currentColor" />
                </svg>
              </div>
            )}
          </div>
        </SlotDropZone>
      </div>
    );
  };

  // Render switch slot with individual drop zone - UPDATED to handle hardcoded icons
  const renderSwitchSlot = (slotIndex: number) => {
    const icon = icons[slotIndex];
    const hardcodedIcon = layoutConfig.slotIcons?.[slotIndex];

    return (
      <SlotDropZone key={`slot-${slotIndex}`} slotIndex={slotIndex}>
        <div
          className={`
            relative flex items-center justify-center 
            rounded-xl ${layoutConfig.slotSize} group
            transition-all duration-100
            ${
              icon || hardcodedIcon
                ? glass === "White"
                  ? "bg-black/10 border-2 border-black/20 shadow-lg"
                  : "bg-white/10 border-2 border-white/20 shadow-lg"
                : glass === "White"
                  ? "border-2 border-black/10 border-dashed hover:border-black/30"
                  : "border-2 border-white/10 border-dashed hover:border-white/30"
            }
          `}
        >
          {/* Show hardcoded icon if present and no user icon */}
          {hardcodedIcon && !icon ? (
            <div className={`flex flex-col items-center justify-center ${glass === "White" ? "text-black/90" : "text-white/90"}`}>
              <img 
                src={hardcodedIcon} 
                alt={`Curtain ${slotIndex + 1}`}
                className="w-8 h-8 object-contain mb-1"
                style={{ filter: glass === "White" ? "brightness(0)" : "brightness(0) invert(1)" }}
              />
              <span className="text-[8px] font-medium text-center">
                CURTAIN {slotIndex + 1}
              </span>
            </div>
          ) : icon ? (
            <>
              <SortableIconSlot
                id={icon.instanceId}
                icon={icon}
                slotIndex={slotIndex}
                onRemove={handleDeleteIcon}
              />
              {/* Hover delete button */}
              <button
                onClick={() => handleDeleteIcon(icon.instanceId)}
                className="absolute -top-2 -right-2 w-6 h-6 bg-red-500 hover:bg-red-600 text-white rounded-full flex items-center justify-center shadow-lg opacity-0 group-hover:opacity-100 transition-all duration-200 z-20 border-2 border-white"
                aria-label={`Delete ${icon.name || "icon"}`}
              >
                <X className="w-3 h-3" />
              </button>
            </>
          ) : (
            <div className={`flex flex-col items-center ${glass === "White" ? "text-black/30" : "text-white/30"}`}>
              <Zap className="w-4 h-4 mb-1" />
              <span className="text-[10px] font-medium">{slotIndex + 1}</span>
            </div>
          )}
        </div>
      </SlotDropZone>
    );
  };

  // NEW: Render main grid with proper handling for curtain mode
  const renderMainGrid = () => {
    const totalSlots = layoutConfig.isFullCurtainMode 
      ? layoutConfig.curtainSlots 
      : layoutConfig.switchSlots;

    return (
      <div
        className={`grid ${layoutConfig.cols} ${layoutConfig.rows} ${layoutConfig.gap} place-items-center`}
      >
        {Array.from({ length: totalSlots }, (_, index) => {
          if (layoutConfig.isFullCurtainMode) {
            // In full curtain mode, render hardcoded curtain icons
            const hardcodedIcon = layoutConfig.slotIcons?.[index];
            return (
              <div
                key={`curtain-${index}`}
                className={`relative flex items-center justify-center rounded-xl w-[60px] h-[60px] border-2 shadow-lg ${
                  glass === "White" 
                    ? "bg-black/10 border-black/20" 
                    : "bg-white/10 border-white/20"
                }`}
              >
                <div className={`flex flex-col items-center justify-center ${glass === "White" ? "text-black/90" : "text-white/90"}`}>
                  <img 
                    src={hardcodedIcon} 
                    alt={`Curtain ${index + 1}`}
                    className="w-8 h-8 object-contain mb-1"
                    style={{ filter: glass === "White" ? "brightness(0)" : "brightness(0) invert(1)" }}
                  />
                  <span className="text-[8px] font-medium text-center">
                    CURTAIN {index + 1}
                  </span>
                </div>
              </div>
            );
          } else {
            // Regular switch slots
            return renderSwitchSlot(index);
          }
        })}
      </div>
    );
  };

  // Count active (non-null) icons
  const activeIconCount = icons.filter(icon => icon !== null).length;

  return (


<div 
  ref={ref}
  data-panel-container="true"
          style={{
          backgroundImage: selectedWall ? `url(${selectedWall})` : "none",
          backgroundSize: "cover",
          backgroundPosition: "center",
          backgroundColor: selectedWall ? "transparent" : "#ADD8E6", // Light blue fallback
        }}

  className="space-y-8 flex flex-col items-center justify-center min-h-screen p-8"
>
   <div className="grid grid-cols-7 gap-4">
        {wallOptions.map((wall, index) => (
          <button
            key={index}
            onClick={() => setSelectedWall(wall)}
            className={`w-10 h-10 rounded-3xl overflow-hidden border-2 transition 
              ${selectedWall === wall ? "border-blue-500" : "border-transparent"}`}
          >
            <img src={wall} alt={`Wall ${index + 1}`} className="w-full h-full object-cover" />
          </button>
        ))}
      </div>


      {/* Panel Info */}
      <div className="flex flex-wrap gap-3 items-center justify-center">
        {/* <Badge variant="outline" className="px-3 py-1 text-sm font-medium">
          {size} Module
        </Badge>
        <Badge variant="outline" className="px-3 py-1 text-sm font-medium">
          {variant}
        </Badge>
        <Badge variant="outline" className="px-3 py-1 text-sm font-medium">
          {glass} Glass
        </Badge>
        <Badge variant="outline" className="px-3 py-1 text-sm font-medium">
          {frame} Frame
        </Badge> */}

        {/* Display component counts
        {layoutConfig.isFullCurtainMode ? (
          <Badge variant="secondary" className="px-3 py-1 text-sm font-medium text-purple-600">
            {layoutConfig.curtainSlots} Curtains (Fixed)
          </Badge>
        ) : (
          <Badge variant="secondary" className="px-3 py-1 text-sm font-medium">
            {layoutConfig.switchSlots} Switches ({icons.slice(0, layoutConfig.switchSlots).filter(icon => icon !== null).length} filled)
          </Badge>
        )} */}
        
        {/* {layoutConfig.curtainSlots && layoutConfig.curtainSlots > 0 && !layoutConfig.isFullCurtainMode && (
          <Badge variant="secondary" className="px-3 py-1 text-sm font-medium text-purple-600">
            {layoutConfig.curtainSlots} Curtains
          </Badge>
        )}
        
        {layoutConfig.fanSlots && layoutConfig.fanSlots > 0 && (
          <Badge variant="secondary" className="px-3 py-1 text-sm font-medium text-blue-600">
            Fan Controls (4 slots)
          </Badge>
        )}
         */}
        {/* {layoutConfig.plugSlots && layoutConfig.plugSlots > 0 && (
          <Badge variant="secondary" className="px-3 py-1 text-sm font-medium text-amber-600">
            {layoutConfig.plugSlots} Plug
          </Badge>
        )*/}

        {activeIconCount > 0 && !layoutConfig.isFullCurtainMode && (
          <Button
            variant="outline"
            size="sm"
            onClick={handleClearAllIcons}
            className="flex items-center gap-2 text-red-600 hover:text-red-700 hover:border-red-300 ml-4"
          >
            <Trash2 className="w-4 h-4" />
            Clear All
          </Button>
        )} 
      </div>

      {/* Touch Panel */}
      <div className="relative">
        {/* Outer frame/bezel */}
        <div className={`rounded-[2rem] ${frameStyle} border-none p-2 shadow-2xl`}>
          <Card className="rounded-[1.5rem] overflow-hidden border-0 shadow-inner">
            <div
              className={`
                relative rounded-[1.5rem] overflow-hidden
                flex items-center justify-center
                ${layoutConfig.panelSize}
                transition-all duration-200
              `}
              style={{ background: glassEffect.background }}
            >
              <div
                className="absolute top-0 left-0 w-full h-full pointer-events-none rounded-[1.5rem]"
                style={{ background: glassEffect.overlay }}
              />

              {/* Subtle grid pattern */}
              <div
                className="absolute inset-0 opacity-5"
                style={{
                  backgroundImage: `
                    linear-gradient(90deg, ${glass === "White" ? "rgba(0,0,0,0.1)" : "rgba(255,255,255,0.1)"} 1px, transparent 1px),
                    linear-gradient(${glass === "White" ? "rgba(0,0,0,0.1)" : "rgba(255,255,255,0.1)"} 1px, transparent 1px)
                  `,
                  backgroundSize: "20px 20px",
                }}
              />

              {/* Main content area with switches and other components */}
              <SortableContext
                items={icons.filter(icon => icon !== null).map((icon) => icon!.instanceId)}
                strategy={rectSortingStrategy}
              >
                <div
                  className={`w-full h-full ${layoutConfig.padding} flex items-center justify-center relative z-10`}
                >
                  <div className="flex items-center justify-center">
                    {/* Main Grid - Updated to use renderMainGrid */}
                    {renderMainGrid()}

                    {/* Curtain Section - Only show if not in full curtain mode */}
                    {!layoutConfig.isFullCurtainMode && renderCurtainSection()}

                    {/* Fan Section */}
                    {renderFanSection()}

                    {/* Plug Section */}
                    {renderPlugSection()}
                  </div>
                </div>
              </SortableContext>

              {/* Brand logo */}
              <div className="absolute bottom-4 right-4 z-10">
                <img
                  src="https://www.autosync.co.in/lovable-uploads/brand/961be73e-48dc-42ec-9d70-cefd0978b603.png"
                  alt="AutoSync Logo"
                  className="w-15 h-7 object-contain opacity-60"
                  style={{ filter: glass === "White" ? "brightness(0)" : "brightness(0) invert(1)" }}
                />
              </div>
            </div>
          </Card>
        </div>
      </div>
    </div>
  );
});

TouchPanelPreview.displayName = "TouchPanelPreview";

export default TouchPanelPreview