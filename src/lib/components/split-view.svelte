<script lang="ts">
  import * as Resizable from "$lib/components/ui/resizable";
  import { Checkbox } from "$lib/components/ui/checkbox/index.js";
  import { Label } from "$lib/components/ui/label/index.js";
  import * as Dialog from "$lib/components/ui/dialog/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import ChevronDown from "lucide-svelte/icons/chevron-down";
  import ChevronRight from "lucide-svelte/icons/chevron-right";
  import Pencil from "lucide-svelte/icons/pencil";
  import Check from "lucide-svelte/icons/check";
  import X from "lucide-svelte/icons/x";
  import ChevronUp from "lucide-svelte/icons/chevron-up";
  import Plus from "lucide-svelte/icons/plus";
  import * as Collapsible from "$lib/components/ui/collapsible/index.js";
  import Resume from "$lib/components/resume.svelte";
  import { Book, Download, HardDriveDownloadIcon } from "lucide-svelte";
  
  // Import libraries for client-side export
  import html2canvas from 'html2canvas';
  import jsPDF from 'jspdf';
  // Remove the problematic html-to-docx import for now
  import fileSaver from 'file-saver';
    import ChevronLeft from "@lucide/svelte/icons/chevron-left";
    import ArrowLeft from "@lucide/svelte/icons/arrow-left";
  const { saveAs } = fileSaver;

  type Props = {
    data: any;
    onDataUpdate?: (newData: any) => void;
  };

  let { data, onDataUpdate }: Props = $props();
  let showDocxInstructions = $state(false); // New state for showing instructions

  let isMobile = $state(false);
  let selectedFields = $state(new Set<string>());
  let expandedSections = $state(new Set<string>(['basics'])); // Start with basics expanded
  let editingFields = $state(new Set<string>());
  let editValues = $state(new Map<string, string>());

  // Download dialog state
  let downloadDialogOpen = $state(false);
  let selectedPaperSize = $state("A4");

  // Helper function to update data and notify parent
  function updateData(newData: any) {
    if (onDataUpdate) {
      onDataUpdate(newData);
    } else {
      // Fallback: direct mutation with warning suppression
      Object.assign(data, newData);
    }
  }

  function convertModernColorsToRGB(element: Element) {
  const computedStyle = window.getComputedStyle(element);
  const rgbRegex = /rgb\([\d\s,]+\)/;
  
  // Create a style object with converted colors
  const convertedStyles: Record<string, string> = {};
  
  // Convert common properties that might use oklch
  ['color', 'background-color', 'border-color', 'border-top-color', 'border-right-color', 'border-bottom-color', 'border-left-color'].forEach(prop => {
    const value = computedStyle.getPropertyValue(prop);
    if (value && !rgbRegex.test(value) && value !== 'transparent' && value !== 'none') {
      // Create a temporary element to get the computed RGB value
      const temp = document.createElement('div');
      temp.style.position = 'absolute';
      temp.style.visibility = 'hidden';
      temp.style.color = value;
      document.body.appendChild(temp);
      const rgbValue = window.getComputedStyle(temp).color;
      document.body.removeChild(temp);
      convertedStyles[prop] = rgbValue;
    }
  });
  
  return convertedStyles;
}

  // Helper function to check if a section is empty
  function isSectionEmpty(sectionKey: string, sectionValue: any): boolean {
    if (sectionKey === 'basics') {
      // Basics is never empty as long as it exists
      return false;
    }
    
    if (Array.isArray(sectionValue)) {
      return sectionValue.length === 0;
    }
    
    if (typeof sectionValue === 'object' && sectionValue !== null) {
      return Object.keys(sectionValue).length === 0;
    }
    
    return !sectionValue;
  }

  // Define proper type for resume data
  type ResumeData = {
    basics: Record<string, any>;
    work: any[];
    education: any[];
    skills: any[];
    awards: any[];
    certificates: any[];
    languages: any[];
    publications: any[];
    projects: any[];
    volunteer: any[];
    interests: any[];
    references: any[];
  };

  // Generate resume data from selected fields - make this reactive
  let resumeData = $derived.by(() => {
    if (!data) return null;

    const result: ResumeData = {
      basics: {},
      work: [],
      education: [],
      skills: [],
      awards: [],
      certificates: [],
      languages: [],
      publications: [],
      projects: [],
      volunteer: [],
      interests: [],
      references: []
    };

    // Convert Set to Array to ensure reactivity
    const selectedArray = Array.from(selectedFields);
    
    console.log('Processing selectedArray:', selectedArray);

    // Process selected fields
    selectedArray.forEach(fieldPath => {
      const pathParts = fieldPath.split('.');
      console.log('Processing fieldPath:', fieldPath, 'pathParts:', pathParts);
      
      if (pathParts[0] === 'basics' && pathParts.length === 2) {
        // Basic fields like basics.name, basics.email - only if basics section is selected
        if (selectedArray.includes('basics')) {
          const field = pathParts[1];
          console.log('Processing basic field:', field, 'value:', data.basics?.[field]);
          if (data.basics && data.basics[field] !== undefined) {
            result.basics[field] = data.basics[field];
            console.log('Added basic field:', field, 'value:', result.basics[field]);
          }
        }
      } else if (pathParts[0] === 'basics' && pathParts[1] === 'profiles' && pathParts.length === 3) {
        // Profile links like basics.profiles.0 - only if basics section is selected
        if (selectedArray.includes('basics')) {
          const index = parseInt(pathParts[2]);
          console.log('Processing profile:', index, 'profile:', data.basics?.profiles?.[index]);
          if (data.basics?.profiles?.[index]) {
            if (!result.basics.profiles) result.basics.profiles = [];
            result.basics.profiles = [...result.basics.profiles, data.basics.profiles[index]];
            console.log('Added profile:', result.basics.profiles);
          }
        }
      } else if (pathParts.length === 2) {
        // Array items like work.0, education.1 - only if the section is selected
        const section = pathParts[0] as keyof ResumeData;
        const index = parseInt(pathParts[1]);
        
        // Only add individual items if the section is selected
        if (selectedArray.includes(section) && 
            data[section] && Array.isArray(data[section]) && data[section][index] && Array.isArray(result[section])) {
          (result[section] as any[]) = [...(result[section] as any[]), data[section][index]];
          console.log('Added individual item to', section, ':', data[section][index]);
        }
      } else if (pathParts.length === 1) {
        // Entire sections - this enables the section for individual items
        const section = pathParts[0] as keyof ResumeData;
        console.log('Section', section, 'is enabled for individual items');
      }
    });

    console.log('Final result.basics:', result.basics);
    console.log('Final result:', result);
    return result;
  });

  // Create contacts array for resume header
  function generateContacts(basics: any) {
    const contacts = [];
    
    if (basics?.email) {
      contacts.push({
        icon: "email",
        label: basics.email,
        href: `mailto:${basics.email}`
      });
    }
    
    if (basics?.phone) {
      contacts.push({
        icon: "@",
        label: basics.phone
      });
    }

    if (basics?.profiles) {
      basics.profiles.forEach((profile: { network: string; url: any; }) => {
        if (profile.network === 'LinkedIn') {
          contacts.push({
            icon: "@",
            label: "LinkedIn",
            href: profile.url
          });
        } else {
          contacts.push({
            icon: "web",
            label: profile.url || profile.network,
            href: profile.url
          });
        }
      });
    }

    return contacts;
  }

  $effect(() => {
    if (data) {
      const defaultSelected = new Set<string>();
      
      // Select main sections by default, but be more selective
      Object.keys(data).forEach(sectionKey => {
        if (sectionKey !== '$schema' && sectionKey !== 'meta' && data[sectionKey] && !isSectionEmpty(sectionKey, data[sectionKey])) {
          
          // For basics, select the section and individual fields
          if (sectionKey === 'basics') {
            const basics = data[sectionKey];
            ['name', 'label', 'email', 'phone', 'summary', 'location'].forEach(field => {
              if (basics[field]) {
                defaultSelected.add(`basics.${field}`);
              }
            });
            
            // Convert main website URL to profile if it exists
            if (basics.url && !basics.profiles?.some((p: { url: any; }) => p.url === basics.url)) {
              if (!basics.profiles) basics.profiles = [];
              basics.profiles.push({
                network: "Website",
                url: basics.url,
                username: ""
              });
              delete basics.url;
              updateData({ ...data });
            }
            
            // Select all profiles by default
            if (basics.profiles) {
              basics.profiles.forEach((_: any, index: any) => {
                defaultSelected.add(`basics.profiles.${index}`);
              });
            }
            
            // Add the basics section checkbox if any basics items are selected
            const hasBasicsItems = ['name', 'label', 'email', 'phone', 'summary', 'location'].some(field => 
              basics[field] && defaultSelected.has(`basics.${field}`)
            ) || (basics.profiles && basics.profiles.length > 0);
            
            if (hasBasicsItems) {
              defaultSelected.add(sectionKey);
            }
          }
          
          // For array sections, select some individual items by default
          else if (Array.isArray(data[sectionKey])) {
            const limitedSections = ['skills', 'awards', 'certificates', 'publications'];
            const limit = limitedSections.includes(sectionKey) ? 3 : data[sectionKey].length;
            
            // Select individual items
            data[sectionKey].slice(0, limit).forEach((_, index) => {
              defaultSelected.add(`${sectionKey}.${index}`);
            });
            
            // Add the section checkbox if any items are selected
            if (limit > 0) {
              defaultSelected.add(sectionKey);
            }
          }
        }
      });
      
      selectedFields = defaultSelected;
    }
  });

  const handleResize = () => {
    isMobile = window.innerWidth < 768;
  };

function toggleField(fieldPath: string) {
  const pathParts = fieldPath.split('.');
  console.log('Toggling field:', fieldPath, 'pathParts:', pathParts);
  
  if (pathParts.length === 1) {
    // Toggling an entire section
    const section = pathParts[0];
    
    if (selectedFields.has(section)) {
      // Uncheck section - remove section AND all its individual items
      selectedFields.delete(section);
      
      // Also remove all individual items in this section
      if (section === 'basics') {
        // Remove all basics fields and profiles
        const fieldsToRemove = Array.from(selectedFields).filter(field => 
          field.startsWith('basics.') && field !== 'basics'
        );
        fieldsToRemove.forEach(field => selectedFields.delete(field));
      } else if (Array.isArray(data[section])) {
        // Remove all individual array items
        data[section].forEach((_, index) => {
          selectedFields.delete(`${section}.${index}`);
        });
      }
    } else {
      // Check section - add section AND all its individual items
      selectedFields.add(section);
      
      // Also add all individual items in this section
      if (section === 'basics') {
        // Add all basics fields
        const basics = data[section];
        ['name', 'label', 'email', 'phone', 'summary', 'location'].forEach(field => {
          if (basics[field]) {
            selectedFields.add(`basics.${field}`);
          }
        });
        
        // Add all profiles
        if (basics.profiles) {
          basics.profiles.forEach((_: any, index: any) => {
            selectedFields.add(`basics.profiles.${index}`);
          });
        }
      } else if (Array.isArray(data[section])) {
        // Add all individual array items
        data[section].forEach((_, index) => {
          selectedFields.add(`${section}.${index}`);
        });
      }
    }
  } else {
    // Toggling individual items
    if (selectedFields.has(fieldPath)) {
      selectedFields.delete(fieldPath);
    } else {
      selectedFields.add(fieldPath);
    }
    
    // Update section checkbox based on whether ANY items are selected
    if (pathParts.length === 2 || (pathParts.length === 3 && pathParts[0] === 'basics' && pathParts[1] === 'profiles')) {
      const section = pathParts[0];
      
      // Check if any items in this section are selected
      const hasSelectedItems = Array.from(selectedFields).some(field => {
        const parts = field.split('.');
        return parts[0] === section && parts.length > 1;
      });
      
      // Update section checkbox based on whether any items are selected
      if (hasSelectedItems && !selectedFields.has(section)) {
        selectedFields.add(section);
      } else if (!hasSelectedItems && selectedFields.has(section)) {
        selectedFields.delete(section);
      }
    }
  }
  
  selectedFields = new Set(selectedFields);
}
  function toggleExpanded(section: string) {
    // Only allow one section to be expanded at a time
    if (expandedSections.has(section)) {
      expandedSections.delete(section);
    } else {
      expandedSections.clear(); // Close all other sections
      expandedSections.add(section);
    }
    expandedSections = new Set(expandedSections);
  }

  function startEdit(fieldPath: string, currentValue: string) {
    editingFields.add(fieldPath);
    editValues.set(fieldPath, currentValue);
    editingFields = new Set(editingFields);
    editValues = new Map(editValues);
  }

  function saveEdit(fieldPath: string) {
    const newValue = editValues.get(fieldPath);
    if (newValue !== undefined) {
      const newData = { ...data };
      
      // Handle nested field updates for languages and profiles
      const pathParts = fieldPath.split('.');
      let obj = newData;
      
      if (pathParts.length === 4 && pathParts[0] === 'languages') {
        // Handle languages.index.field format
        const index = parseInt(pathParts[1]);
        const field = pathParts[2];
        if (newData.languages && newData.languages[index]) {
          newData.languages[index][field] = newValue;
        }
      } else if (pathParts.length === 3 && pathParts[0] === 'basics' && pathParts[1] === 'profiles') {
        // Handle basics.profiles.index format - update the URL
        const index = parseInt(pathParts[2]);
        if (newData.basics.profiles && newData.basics.profiles[index]) {
          newData.basics.profiles[index].url = newValue;
        }
      } else {
        // Handle regular field updates
        for (let i = 0; i < pathParts.length - 1; i++) {
          obj = obj[pathParts[i]];
        }
        obj[pathParts[pathParts.length - 1]] = newValue;
      }
      
      updateData(newData);
    }
    
    editingFields.delete(fieldPath);
    editValues.delete(fieldPath);
    editingFields = new Set(editingFields);
    editValues = new Map(editValues);
  }

  function cancelEdit(fieldPath: string) {
    editingFields.delete(fieldPath);
    editValues.delete(fieldPath);
    editingFields = new Set(editingFields);
    editValues = new Map(editValues);
  }

  function handleKeydown(event: KeyboardEvent, fieldPath: string) {
    if (event.key === 'Enter') {
      event.preventDefault();
      saveEdit(fieldPath);
    } else if (event.key === 'Escape') {
      event.preventDefault();
      cancelEdit(fieldPath);
    }
  }

  function moveItem(sectionKey: string, fromIndex: number, toIndex: number) {
    if (Array.isArray(data[sectionKey])) {
      const newData = { ...data };
      const items = [...newData[sectionKey]];
      const [movedItem] = items.splice(fromIndex, 1);
      items.splice(toIndex, 0, movedItem);
      newData[sectionKey] = items;
      updateData(newData);
    }
  }

  function moveLink(fromIndex: number, toIndex: number) {
    if (!data.basics.profiles || fromIndex < 0 || toIndex < 0 || 
        fromIndex >= data.basics.profiles.length || toIndex >= data.basics.profiles.length ||
        fromIndex === toIndex) return;
    
    const newData = { ...data };
    const profiles = [...newData.basics.profiles];
    const [movedProfile] = profiles.splice(fromIndex, 1);
    profiles.splice(toIndex, 0, movedProfile);
    newData.basics.profiles = profiles;
    updateData(newData);
  }
  
  function addLink() {
    const newData = { ...data };
    if (!newData.basics.profiles) {
      newData.basics.profiles = [];
    }
    
    const newProfile = {
      network: "Website",
      url: "https://example.com",
      username: ""
    };
    
    newData.basics.profiles = [...newData.basics.profiles, newProfile];
    updateData(newData);
    
    // Auto-select the new link
    const newIndex = newData.basics.profiles.length - 1;
    selectedFields.add(`basics.profiles.${newIndex}`);
    selectedFields = new Set(selectedFields);
  }

  function removeLink(index: number) {
    if (!data.basics.profiles || index < 0 || index >= data.basics.profiles.length) return;
    
    // Remove from selected fields first
    selectedFields.delete(`basics.profiles.${index}`);
    
    // Remove the profile
    const newData = { ...data };
    const profiles = [...newData.basics.profiles];
    profiles.splice(index, 1);
    newData.basics.profiles = profiles;
    updateData(newData);
    
    // Update selected fields for remaining profiles (shift indices down)
    const updatedSelected = new Set<string>();
    selectedFields.forEach(field => {
      if (field.startsWith('basics.profiles.')) {
        const fieldIndex = parseInt(field.split('.')[2]);
        if (fieldIndex > index) {
          updatedSelected.add(`basics.profiles.${fieldIndex - 1}`);
        } else if (fieldIndex < index) {
          updatedSelected.add(field);
        }
      } else {
        updatedSelected.add(field);
      }
    });
    selectedFields = updatedSelected;
  }

  function getNewItemTemplate(sectionKey: string) {
    switch (sectionKey) {
      case 'skills':
        return { name: "New Skill", level: "", keywords: [] };
      case 'certificates':
        return { name: "New Certificate", issuer: "Issuer", date: "", url: "" };
      case 'languages':
        return { language: "Language", fluency: "Conversational" };
      default:
        return {};
    }
  }

  function formatSectionName(key: string): string {
    const sectionNames: Record<string, string> = {
      basics: "Personal Information",
      work: "Work Experience", 
      volunteer: "Volunteer Experience",
      education: "Education",
      awards: "Awards & Honors",
      certificates: "Certifications",
      publications: "Publications",
      skills: "Skills",
      languages: "Languages",
      interests: "Interests",
      references: "References",
      projects: "Projects"
    };
    return sectionNames[key] || key.charAt(0).toUpperCase() + key.slice(1);
  }

  function formatItemLabel(item: any, section: string): string {
    switch (section) {
      case 'work':
      case 'volunteer':
        return `${item.position} at ${item.name || item.organization}`;
      case 'education':
        return `${item.institution} ${item.studyType ? `(${item.studyType})` : ''}`.trim();
      case 'awards':
        return item.title;
      case 'certificates':
        return `${item.name} - ${item.issuer}`;
      case 'publications':
        return item.name;
      case 'skills':
        return item.name;
      case 'languages':
        return `${item.language} (${item.fluency})`;
      default:
        return JSON.stringify(item).slice(0, 50) + '...';
    }
  }

  function getWebsiteType(url: string): string {
    if (!url) return 'Website';
    if (url.includes('github.com')) return 'GitHub';
    if (url.includes('dribbble.com')) return 'Dribbble';
    if (url.includes('youtube.com')) return 'YouTube';
    if (url.includes('orcid.org')) return 'ORCID';
    return 'Website';
  }

  function renderBasics(basics: any) {
    const basicFields = [
      { key: 'name', label: 'Full Name', value: basics.name },
      { key: 'label', label: 'Professional Title', value: basics.label },
      { key: 'email', label: 'Email', value: basics.email },
      { key: 'phone', label: 'Phone', value: basics.phone },
      { key: 'summary', label: 'Summary', value: basics.summary },
      { key: 'location', label: 'Location', value: basics.location?.address },
    ];

    return basicFields.filter(field => field.value);
  }

  function renderLinks(basics: any) {
    const links: { key: string; label: string; value: any; fieldPath: string; }[] = [];
    
    // Only add profiles (no separate website URL handling)
    if (basics.profiles) {
      basics.profiles.forEach((profile: { network: string; url: string; username: any; }, index: any) => {
        const network = profile.network || 'Website';
        const url = profile.url || '';
        const label = network === 'LinkedIn' ? 'LinkedIn' : getWebsiteType(url);
        links.push({
          key: `profile-${index}`,
          label,
          value: url || `https://${network.toLowerCase()}.com/${profile.username || ''}`,
          fieldPath: `basics.profiles.${index}`
        });
      });
    }
    
    return links;
  }

  function isTextField(section: string, field?: string): boolean {
    if (section === 'basics') {
      return ['name', 'label', 'email', 'phone', 'summary', 'location'].includes(field || '');
    }
    return false;
  }

  function canReorder(sectionKey: string): boolean {
    return ['skills', 'certificates', 'languages'].includes(sectionKey);
  }

  function canAddItems(sectionKey: string): boolean {
    return false; // Disable adding items for all sections
  }

  function isLanguageEditable(sectionKey: string): boolean {
    return sectionKey === 'languages';
  }

  // Helper function to get resume HTML with embedded styles
  function getResumeHTML(): string {
    const resumeElement = document.querySelector('[data-resume-container]');
    if (!resumeElement) {
      throw new Error('Resume container not found');
    }
    
    // Get computed styles for all elements
    function getElementStyles(element: Element): string {
      const computedStyle = window.getComputedStyle(element);
      let styles = '';
      
      // Copy all computed styles
      for (let i = 0; i < computedStyle.length; i++) {
        const property = computedStyle[i];
        const value = computedStyle.getPropertyValue(property);
        styles += `${property}: ${value}; `;
      }
      
      return styles;
    }

    // Clone the element and apply inline styles
    const clonedElement = resumeElement.cloneNode(true) as Element;
    
    function applyInlineStyles(element: Element, original: Element) {
      element.setAttribute('style', getElementStyles(original));
      
      // Apply styles to all children
      const children = element.children;
      const originalChildren = original.children;
      
      for (let i = 0; i < children.length; i++) {
        if (originalChildren[i]) {
          applyInlineStyles(children[i], originalChildren[i]);
        }
      }
    }
    
    applyInlineStyles(clonedElement, resumeElement);
    
    // Create complete HTML document
    return `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Resume</title>
        <style>
          body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            line-height: 1.5;
            color: #000;
            background: #fff;
            margin: 0;
            padding: 20px;
          }
          
          @media print {
            body { margin: 0; padding: 0; }
            * { -webkit-print-color-adjust: exact !important; }
          }
        </style>
      </head>
      <body>
        ${clonedElement.outerHTML}
      </body>
      </html>
    `;
  }

  // Helper function to download blob
  function downloadBlob(blob: Blob, filename: string) {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  // Generate PDF using html2canvas + jsPDF
async function generatePDF(paperSize: string) {
  try {
    const resumeElement = document.querySelector('[data-resume-container]') as HTMLElement;
    if (!resumeElement) {
      throw new Error('Resume container not found');
    }

    console.log('Generating PDF...');

    const paperSizes = {
      A4: { width: 210, height: 297 },
      Letter: { width: 216, height: 279 }
    };
    
    const { width: paperWidth, height: paperHeight } = paperSizes[paperSize as keyof typeof paperSizes];
    const margin = 10; // 10mm margin on all sides
    const contentWidth = paperWidth - (margin * 2);
    const contentHeight = paperHeight - (margin * 2);
    
    // Create PDF instance
    const pdf = new jsPDF({
      orientation: 'portrait',
      unit: 'mm',
      format: paperSize.toLowerCase() as any
    });

    // Clone and prepare the element
    const clonedElement = resumeElement.cloneNode(true) as HTMLElement;
    
    // Recursively apply RGB conversions
    function applyRGBStyles(element: Element) {
      const convertedStyles = convertModernColorsToRGB(element);
      Object.entries(convertedStyles).forEach(([prop, value]) => {
        (element as HTMLElement).style.setProperty(prop, value, 'important');
      });
      
      // Process children
      Array.from(element.children).forEach(child => applyRGBStyles(child));
    }
    
    applyRGBStyles(clonedElement);
    
    // Set up the content for print
    clonedElement.style.cssText = `
      width: ${contentWidth * 3.78}px !important;
      min-height: auto !important;
      max-height: none !important;
      overflow: visible !important;
      font-size: 14px !important;
      line-height: 1.4 !important;
      color: #000 !important;
      background: #fff !important;
      page-break-inside: avoid !important;
    `;

    // Create temporary container
    const tempContainer = document.createElement('div');
    tempContainer.style.cssText = `
      position: absolute;
      left: -9999px;
      top: 0;
      width: ${contentWidth * 3.78}px;
      background: #fff;
      overflow: visible;
    `;
    tempContainer.appendChild(clonedElement);
    document.body.appendChild(tempContainer);

    // First pass: measure total height
    const totalHeight = tempContainer.scrollHeight;
    const pageHeightPx = contentHeight * 3.78; // Convert mm to px at scale
    const totalPages = Math.ceil(totalHeight / pageHeightPx);

    console.log(`Total height: ${totalHeight}px, Page height: ${pageHeightPx}px, Total pages: ${totalPages}`);

    // Generate each page separately
    for (let pageNum = 0; pageNum < totalPages; pageNum++) {
      if (pageNum > 0) {
        pdf.addPage();
      }

      // Calculate the viewport for this page
      const startY = pageNum * pageHeightPx;
      const endY = Math.min((pageNum + 1) * pageHeightPx, totalHeight);
      const actualPageHeight = endY - startY;

      // Create a container for just this page's content
      const pageContainer = document.createElement('div');
      pageContainer.style.cssText = `
        position: absolute;
        left: -9999px;
        top: 0;
        width: ${contentWidth * 3.78}px;
        height: ${actualPageHeight}px;
        background: #fff;
        overflow: hidden;
      `;

      // Clone the content again for this page
      const pageClonedElement = clonedElement.cloneNode(true) as HTMLElement;
      pageClonedElement.style.cssText = `
        width: ${contentWidth * 3.78}px !important;
        transform: translateY(-${startY}px);
        overflow: visible !important;
        font-size: 14px !important;
        line-height: 1.4 !important;
        color: #000 !important;
        background: #fff !important;
      `;

      pageContainer.appendChild(pageClonedElement);
      document.body.appendChild(pageContainer);

      try {
        // Capture this page
        const canvas = await html2canvas(pageContainer, {
          scale: 3,
          useCORS: true,
          allowTaint: true,
          backgroundColor: '#ffffff',
          width: contentWidth * 3.78,
          height: actualPageHeight,
          logging: false
        });

        // Add to PDF
        const imgData = canvas.toDataURL('image/png', 1.0);
        const pdfHeight = (actualPageHeight * contentWidth) / (contentWidth * 3.78);
        
        pdf.addImage(imgData, 'PNG', margin, margin, contentWidth, pdfHeight);

        console.log(`Page ${pageNum + 1}/${totalPages} rendered`);
      } catch (pageError) {
        console.error(`Error rendering page ${pageNum + 1}:`, pageError);
      }

      // Clean up this page's container
      document.body.removeChild(pageContainer);
    }

    // Clean up main container
    document.body.removeChild(tempContainer);

    pdf.save('resume.pdf');
    console.log('PDF generated successfully');
  } catch (error) {
    console.error('PDF generation failed:', error);
    alert('Failed to generate PDF. Please try again.');
  }
}
  // Simplified DOCX generation using dynamic import to avoid circular dependency
async function generateDOCX(paperSize: string) {
    try {
      console.log('Generating DOCX...');
      
      // Dynamic import to avoid circular dependency issues
      const { default: HTMLtoDOCX } = await import('html-to-docx');
      
      const resumeElement = document.querySelector('[data-resume-container]');
      if (!resumeElement) {
        throw new Error('Resume container not found');
      }

      // Get the HTML content with inline styles
      const htmlContent = getResumeHTML();
      
      // Convert HTML to DOCX using html-to-docx with simplified options
      const docxBuffer = await HTMLtoDOCX(htmlContent, null, {
        font: 'Arial',
        fontSize: 11,
        margins: {
          top: 720,    // 0.5 inch in twips
          right: 720,
          bottom: 720,
          left: 720
        },
        orientation: 'portrait',
        // Add page size based on selection
        ...(paperSize === 'A4' ? {
          width: 11906,  // A4 width in twips
          height: 16838  // A4 height in twips
        } : {
          width: 12240,  // Letter width in twips
          height: 15840  // Letter height in twips
        })
      });

      // Create blob and download
      const blob = new Blob([docxBuffer], { 
        type: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document' 
      });
      
      saveAs(blob, 'resume.docx');
      console.log('DOCX generated successfully');
      downloadDialogOpen = false;
      
    } catch (error) {
      console.error('DOCX generation failed:', error);
      
      // Show instructions UI instead of alert
      showDocxInstructions = true;
    }
  }

  // Handle going back from instructions
  function goBackFromInstructions() {
    showDocxInstructions = false;
  }
  // Export HTML
  function exportHTML() {
    try {
      console.log('Exporting HTML...');
      const html = getResumeHTML();
      const blob = new Blob([html], { type: 'text/html;charset=utf-8' });
      downloadBlob(blob, 'resume.html');
      console.log('HTML exported successfully');
    } catch (error) {
      console.error('HTML export failed:', error);
      alert('Failed to export HTML. Please try again.');
    }
  }

  // Export JSON
  function exportJSON() {
    try {
      console.log('Exporting JSON...');
      const jsonData = JSON.stringify(resumeData, null, 2);
      const blob = new Blob([jsonData], { type: 'application/json;charset=utf-8' });
      downloadBlob(blob, 'resume.json');
      console.log('JSON exported successfully');
    } catch (error) {
      console.error('JSON export failed:', error);
      alert('Failed to export JSON. Please try again.');
    }
  }
    function downloadHTMLFromInstructions() {
    exportHTML();
    downloadDialogOpen = false;
    showDocxInstructions = false;
  }

  async function handleDownload(format: string, paperSize: string) {
    console.log(`Downloading resume as ${format} with ${paperSize} paper size`);
    
    try {
      switch (format) {
        case 'PDF':
          await generatePDF(paperSize);
          break;
        case 'DOCX':
          await generateDOCX(paperSize);
          break;
        case 'HTML':
          exportHTML();
          break;
        case 'JSON':
          exportJSON();
          break;
        default:
          throw new Error(`Unsupported format: ${format}`);
      }
    } catch (error) {
      console.error('Download failed:', error);
      alert(`Failed to download ${format}. Please try again.`);
    }
    
    if (format !== 'DOCX') {
      downloadDialogOpen = false;
    }
  }
</script>

{#snippet sectionRenderer(sectionKey: string, sectionValue: any, idSuffix: string)}
  <div class="space-y-1">
    <div class="flex items-center justify-between py-1">
      <div class="flex items-center space-x-2">
        <Checkbox 
          id="section-{sectionKey}-{idSuffix}"
          checked={selectedFields.has(sectionKey)}
          onCheckedChange={() => toggleField(sectionKey)}
        />
        <Label 
          for="section-{sectionKey}-{idSuffix}" 
          class="text-sm font-medium cursor-pointer"
          onclick={(e) => {
            // If section is collapsed, expand it instead of toggling checkbox
            if (!expandedSections.has(sectionKey) && ((Array.isArray(sectionValue) && sectionValue.length > 0) || (sectionKey === 'basics'))) {
              e.preventDefault();
              toggleExpanded(sectionKey);
            }
            // If expanded, let the default label behavior work (toggle checkbox)
          }}
        >
          {formatSectionName(sectionKey)}
        </Label>
      </div>
      <div class="flex items-center space-x-1">
        {#if (Array.isArray(sectionValue) && sectionValue.length > 0) || (sectionKey === 'basics')}
          <button 
            onclick={() => toggleExpanded(sectionKey)}
            class="p-1 hover:bg-gray-100 rounded"
          >
            {#if expandedSections.has(sectionKey)}
              <ChevronDown class="h-3 w-3" />
            {:else}
              <ChevronRight class="h-3 w-3" />
            {/if}
          </button>
        {/if}
      </div>
    </div>

    {#if expandedSections.has(sectionKey)}
      <div class="pl-6 space-y-1">
        {#if sectionKey === 'basics'}
          {#each renderBasics(sectionValue) as field}
            <div class="flex items-start space-x-2 py-1">
              <Checkbox 
                id="basics-{field.key}-{idSuffix}"
                checked={selectedFields.has(`basics.${field.key}`)}
                onCheckedChange={() => toggleField(`basics.${field.key}`)}
              />
              <div class="flex-1 min-w-0">
                <div class="flex items-center space-x-1">
                  <Label for="basics-{field.key}-{idSuffix}" class="text-xs font-medium">
                    {field.label}:
                  </Label>
                  {#if isTextField(sectionKey, field.key)}
                    <button 
                      onclick={() => startEdit(`basics.${field.key}`, field.value)}
                      class="p-0.5 hover:bg-gray-100 rounded"
                    >
                      <Pencil class="h-2.5 w-2.5" />
                    </button>
                  {/if}
                </div>
                {#if editingFields.has(`basics.${field.key}`)}
                  <div class="flex items-center space-x-1 mt-1">
                    <input 
                      type="text" 
                      value={editValues.get(`basics.${field.key}`) || field.value}
                      oninput={(e) => {
                        const target = e.target as HTMLInputElement;
                        editValues.set(`basics.${field.key}`, target.value);
                        editValues = new Map(editValues);
                      }}
                      onkeydown={(e) => handleKeydown(e, `basics.${field.key}`)}
                      class="text-xs border rounded px-1 py-0.5 flex-1"
                    />
                    <button 
                      onclick={() => saveEdit(`basics.${field.key}`)}
                      class="p-0.5 hover:bg-green-100 rounded"
                    >
                      <Check class="h-2.5 w-2.5 text-green-600" />
                    </button>
                    <button 
                      onclick={() => cancelEdit(`basics.${field.key}`)}
                      class="p-0.5 hover:bg-red-100 rounded"
                    >
                      <X class="h-2.5 w-2.5 text-red-600" />
                    </button>
                  </div>
                {:else}
                  <p class="text-xs text-gray-600 truncate">{field.value}</p>
                {/if}
              </div>
            </div>
          {/each}
          
          <!-- Links subsection -->
          {#if renderLinks(sectionValue).length > 0 || sectionValue.profiles}
            <div class="mt-2">
              <Label class="text-xs font-medium text-gray-800 mb-1 block">Links:</Label>
              
              <!-- Links container with consistent right-aligned buttons -->
              <div class="pl-3 space-y-1">
                {#each renderLinks(sectionValue) as link, index}
                  <div class="flex items-start py-1">
                    <div class="flex items-start space-x-2 flex-1 min-w-0">
                      <Checkbox 
                        id="links-{link.key}-{idSuffix}"
                        checked={selectedFields.has(link.fieldPath)}
                        onCheckedChange={() => toggleField(link.fieldPath)}
                      />
                      <div class="flex-1 min-w-0">
                        <div class="flex items-center space-x-1">
                          <Label for="links-{link.key}-{idSuffix}" class="text-xs font-medium">
                            {link.label}:
                          </Label>
                          <button 
                            onclick={() => startEdit(link.fieldPath, link.value)}
                            class="p-0.5 hover:bg-gray-100 rounded"
                          >
                            <Pencil class="h-2.5 w-2.5" />
                          </button>
                        </div>
                        {#if editingFields.has(link.fieldPath)}
                          <div class="flex items-center space-x-1 mt-1">
                            <input 
                              type="text" 
                              value={editValues.get(link.fieldPath) || link.value}
                              oninput={(e) => {
                                const target = e.target as HTMLInputElement;
                                editValues.set(link.fieldPath, target.value);
                                editValues = new Map(editValues);
                              }}
                              onkeydown={(e) => handleKeydown(e, link.fieldPath)}
                              class="text-xs border rounded px-1 py-0.5 flex-1"
                            />
                            <button 
                              onclick={() => saveEdit(link.fieldPath)}
                              class="p-0.5 hover:bg-green-100 rounded"
                            >
                              <Check class="h-2.5 w-2.5 text-green-600" />
                            </button>
                            <button 
                              onclick={() => cancelEdit(link.fieldPath)}
                              class="p-0.5 hover:bg-red-100 rounded"
                            >
                              <X class="h-2.5 w-2.5 text-red-600" />
                            </button>
                          </div>
                        {:else}
                          <p class="text-xs text-gray-600 truncate">{link.value}</p>
                        {/if}
                      </div>
                    </div>
                    
                    <!-- Right-aligned action buttons (move stacked, remove separate) -->
                    <div class="flex items-center space-x-1 ml-4">
                      <div class="flex flex-col space-y-0.5">
                        <button 
                          onclick={() => moveLink(index, index - 1)}
                          disabled={index === 0}
                          class="p-0.5 hover:bg-gray-100 rounded disabled:opacity-50"
                          title="Move up"
                        >
                          <ChevronUp class="h-2.5 w-2.5" />
                        </button>
                        <button 
                          onclick={() => moveLink(index, index + 1)}
                          disabled={index === renderLinks(sectionValue).length - 1}
                          class="p-0.5 hover:bg-gray-100 rounded disabled:opacity-50"
                          title="Move down"
                        >
                          <ChevronDown class="h-2.5 w-2.5" />
                        </button>
                      </div>
                      <button 
                        onclick={() => removeLink(index)}
                        class="p-0.5 hover:bg-red-100 rounded"
                        title="Remove link"
                      >
                        <X class="h-2.5 w-2.5 text-red-600" />
                      </button>
                    </div>
                  </div>
                {/each}
                
                <!-- Add Links button below all entries -->
                <div class="pt-1">
                  <button 
                    onclick={() => addLink()}
                    class="flex items-center space-x-1 text-xs text-black hover:bg-gray-100 px-1 py-0.5 rounded"
                  >
                    <Plus class="h-3 w-3" />
                    <span>Add Links</span>
                  </button>
                </div>
              </div>
            </div>
          {:else}
            <!-- Show Add Links button even when no links exist -->
            <div class="mt-2">
              <Label class="text-xs font-medium text-gray-800 mb-1 block">Links:</Label>
              <div class="pl-3">
                <button 
                  onclick={() => addLink()}
                  class="flex items-center space-x-1 text-xs text-black hover:bg-gray-100 px-1 py-0.5 rounded"
                >
                  <Plus class="h-3 w-3" />
                  <span>Add Links</span>
                </button>
              </div>
            </div>
          {/if}
        {:else if Array.isArray(sectionValue)}
          {#each sectionValue as item, index}
            <div class="flex items-start py-1">
              <div class="flex items-start space-x-2 flex-1 min-w-0">
                <Checkbox 
                  id="{sectionKey}-{index}-{idSuffix}"
                  checked={selectedFields.has(`${sectionKey}.${index}`)}
                  onCheckedChange={() => toggleField(`${sectionKey}.${index}`)}
                />
                <div class="flex-1 min-w-0">
                  {#if isLanguageEditable(sectionKey)}
                    <!-- Editable language fields - inline layout -->
                    <div class="space-y-1">
                      <!-- Language field -->
                      {#if editingFields.has(`${sectionKey}.${index}.language`)}
                        <div class="flex items-center space-x-1">
                          <Label class="text-xs font-medium">Language:</Label>
                          <input 
                            type="text" 
                            value={editValues.get(`${sectionKey}.${index}.language`) || item.language}
                            oninput={(e) => {
                              const target = e.target as HTMLInputElement;
                              editValues.set(`${sectionKey}.${index}.language`, target.value);
                              editValues = new Map(editValues);
                            }}
                            onkeydown={(e) => handleKeydown(e, `${sectionKey}.${index}.language`)}
                            class="text-xs border rounded px-1 py-0.5 flex-1"
                          />
                          <button 
                            onclick={() => saveEdit(`${sectionKey}.${index}.language`)}
                            class="p-0.5 hover:bg-green-100 rounded"
                          >
                            <Check class="h-2.5 w-2.5 text-green-600" />
                          </button>
                          <button 
                            onclick={() => cancelEdit(`${sectionKey}.${index}.language`)}
                            class="p-0.5 hover:bg-red-100 rounded"
                          >
                            <X class="h-2.5 w-2.5 text-red-600" />
                          </button>
                        </div>
                      {:else}
                        <div class="flex items-center space-x-1">
                          <Label for="{sectionKey}-{index}-{idSuffix}" class="text-xs font-medium">Language:</Label>
                          <p class="text-xs text-gray-600">{item.language}</p>
                          <button 
                            onclick={() => startEdit(`${sectionKey}.${index}.language`, item.language)}
                            class="p-0.5 hover:bg-gray-100 rounded"
                          >
                            <Pencil class="h-2.5 w-2.5" />
                          </button>
                        </div>
                      {/if}
                      
                      <!-- Fluency field -->
                      {#if editingFields.has(`${sectionKey}.${index}.fluency`)}
                        <div class="flex items-center space-x-1">
                          <Label class="text-xs font-medium">Fluency:</Label>
                          <input 
                            type="text" 
                            value={editValues.get(`${sectionKey}.${index}.fluency`) || item.fluency}
                            oninput={(e) => {
                              const target = e.target as HTMLInputElement;
                              editValues.set(`${sectionKey}.${index}.fluency`, target.value);
                              editValues = new Map(editValues);
                            }}
                            onkeydown={(e) => handleKeydown(e, `${sectionKey}.${index}.fluency`)}
                            class="text-xs border rounded px-1 py-0.5 flex-1"
                          />
                          <button 
                            onclick={() => saveEdit(`${sectionKey}.${index}.fluency`)}
                            class="p-0.5 hover:bg-green-100 rounded"
                          >
                            <Check class="h-2.5 w-2.5 text-green-600" />
                          </button>
                          <button 
                            onclick={() => cancelEdit(`${sectionKey}.${index}.fluency`)}
                            class="p-0.5 hover:bg-red-100 rounded"
                          >
                            <X class="h-2.5 w-2.5 text-red-600" />
                          </button>
                        </div>
                      {:else}
                        <div class="flex items-center space-x-1">
                          <Label class="text-xs font-medium">Fluency:</Label>
                          <p class="text-xs text-gray-600">{item.fluency}</p>
                          <button 
                            onclick={() => startEdit(`${sectionKey}.${index}.fluency`, item.fluency)}
                            class="p-0.5 hover:bg-gray-100 rounded"
                          >
                            <Pencil class="h-2.5 w-2.5" />
                          </button>
                        </div>
                      {/if}
                    </div>
                  {:else}
                    <!-- Non-editable items (skills, certificates, etc.) -->
                    <Label for="{sectionKey}-{index}-{idSuffix}" class="text-xs font-medium">
                      {formatItemLabel(item, sectionKey)}
                    </Label>
                  {/if}
                </div>
              </div>
              
              <!-- Right-aligned reorder buttons (stacked) for all array items -->
              {#if canReorder(sectionKey)}
                <div class="flex flex-col space-y-0.5 ml-4">
                  <button 
                    onclick={() => moveItem(sectionKey, index, index - 1)}
                    disabled={index === 0}
                    class="p-0.5 hover:bg-gray-100 rounded disabled:opacity-50"
                    title="Move up"
                  >
                    <ChevronUp class="h-2.5 w-2.5" />
                  </button>
                  <button 
                    onclick={() => moveItem(sectionKey, index, index + 1)}
                    disabled={index === sectionValue.length - 1}
                    class="p-0.5 hover:bg-gray-100 rounded disabled:opacity-50"
                    title="Move down"
                  >
                    <ChevronDown class="h-2.5 w-2.5" />
                  </button>
                </div>
              {/if}
            </div>
          {/each}
        {/if}
      </div>
    {/if}
  </div>
{/snippet}

<svelte:window onresize={handleResize} />

<!-- Update the Resume component container to include data attribute -->
<div class="h-dvh w-full">
  {#if isMobile}
    <!-- Mobile layout remains the same -->
    <div class="h-full overflow-auto p-3 space-y-2">
      {#if data}
        {#each Object.entries(data) as [sectionKey, sectionValue]}
          {#if sectionKey !== '$schema' && sectionKey !== 'meta' && sectionValue && !isSectionEmpty(sectionKey, sectionValue)}
            {@render sectionRenderer(sectionKey, sectionValue, 'mobile')}
          {/if}
        {/each}
      {/if}
    </div>
  {:else}
    <Resizable.PaneGroup direction="horizontal">
      <Resizable.Pane>
        <div class="h-full overflow-auto p-3 space-y-2">
          <h3 class="text-base font-semibold mb-3">Resume Sections</h3>
          {#if data}
            {#each Object.entries(data) as [sectionKey, sectionValue]}
              {#if sectionKey !== '$schema' && sectionKey !== 'meta' && sectionValue && !isSectionEmpty(sectionKey, sectionValue)}
                {@render sectionRenderer(sectionKey, sectionValue, 'desktop')}
              {/if}
            {/each}
          {/if}
        </div>
      </Resizable.Pane>
      <Resizable.Handle withHandle />
      <Resizable.Pane>
        <div class="h-full flex flex-col">
          <!-- Resume Preview Section -->
          <div class="flex-1 overflow-auto">
            {#if resumeData && resumeData.basics && Object.keys(resumeData.basics).length > 0}
              <div data-resume-container>
                <Resume 
                  data={resumeData}
                  contacts={generateContacts(resumeData.basics)}
                />
              </div>
            {:else}
              <div class="p-3">
                <h3 class="text-base font-semibold mb-3">Resume Preview</h3>
                <p class="text-gray-500 text-sm">Select items from the left panel to see your resume preview</p>
              </div>
            {/if}
          </div>
          
          <!-- Download Button Section - remains the same -->
          <div class="border-t bg-gray-50 p-3">
            <Dialog.Root bind:open={downloadDialogOpen}>
              <Dialog.Trigger>
                <Button 
                  class="w-full mx-auto max-w-lg flex text-center items-center justify-center gap-2"
                  disabled={!resumeData || !resumeData.basics || Object.keys(resumeData.basics).length === 0}
                >
                  <Download class="w-[17px] h-[17px]"/> Download
                </Button>
              </Dialog.Trigger>
              <Dialog.Content class="sm:max-w-[425px]">
                {#if !showDocxInstructions}
                  <!-- Normal download dialog -->
                    <Dialog.Title>Download Resume</Dialog.Title>
                  <div class="grid gap-6 py-4">
                    <!-- Paper Size Selection with Toggle Buttons -->
                    <div class="space-y-3">
                      <Label class="text-sm font-medium">Paper Size</Label>
                      <div class="flex gap-2">
                        <Button
                          variant={selectedPaperSize === "A4" ? "default" : "outline"}
                          size="sm"
                          onclick={() => selectedPaperSize = "A4"}
                          class="flex-1 cursor-pointer"
                        >
                          A4
                        </Button>
                        <Button
                          variant={selectedPaperSize === "Letter" ? "default" : "outline"}
                          size="sm"
                          onclick={() => selectedPaperSize = "Letter"}
                          class="flex-1 cursor-pointer"
                        >
                          Letter
                        </Button>
                      </div>
                    </div>

                    <div class="space-y-3">
                      <Label class="text-sm font-medium inline ">Download</Label>

                      <div class="flex gap-2 pt-2">
                                                <Button 
                          onclick={() => handleDownload('PDF', selectedPaperSize)}
                          class="flex-1 cursor-pointer"
                        >
                          <Download/> Save as PDF
                        </Button>
                                              <Button 
                        onclick={() => handleDownload('DOCX', selectedPaperSize)}
                        class="flex-1 cursor-pointer"
                      >
                        <Book /> DOCX Instructions
                      </Button>
                      </div>

                      <Button 
                        variant="outline" 
                        onclick={() => downloadDialogOpen = false}
                        class="w-full cursor-pointer"
                      >
                        Cancel
                      </Button>
                                            <div class="text-center inline">
                        <p class="text-xs text-gray-500 -mb-2">or export as
                          <button 
                            onclick={() => handleDownload('HTML', selectedPaperSize)}
                            class="cursor-pointer hover:opacity-80 text-xs text-gray-500 underline hover:text-gray-700"
                          >
                            HTML
                          </button> or 
                          <button 
                            onclick={() => handleDownload('JSON', selectedPaperSize)}
                            class="cursor-pointer hover:opacity-80 text-xs text-gray-500 underline hover:text-gray-700"
                          >
                            JSON
                          </button>
                        </p>
                      </div>

                    </div>
                  </div>
                {:else}
                  <Dialog.Header>
                    <div class="flex items-center gap-2">
                      <button 
                        onclick={goBackFromInstructions}
                        class="p-1 hover:bg-gray-100 rounded-full"
                        title="Go back"
                      >
                        <ArrowLeft class="h-4 w-4" />
                      </button>
                      <Dialog.Title>DOCX Instructions</Dialog.Title>
                    </div>
                    <Dialog.Description>
                      Direct DOCX export isn't something this website has. You can still easily get your resume as a Word document:
                    </Dialog.Description>
                  </Dialog.Header>
                  
                  <div class="py-4 space-y-4">
                    <div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
                      <h4 class="font-semibold text-blue-900 mb-2">Method 1: HTML → Word (Recommended)</h4>
                      <ol class="text-sm text-blue-800 space-y-1 list-decimal list-inside">
                        <li>Download as HTML using the button below</li>
                        <li>Open the HTML file in Microsoft Word or LibreOffice Writer</li>
                        <li>Save as DOCX from within Word/Writer</li>
                      </ol>
                    </div>
                    
                    <div class="bg-green-50 border border-green-200 rounded-lg p-4">
                      <h4 class="font-semibold text-green-900 mb-2">Method 2: PDF → Word</h4>
                      <ol class="text-sm text-green-800 space-y-1 list-decimal list-inside">
                        <li>Download as PDF, which can be opened and edited in modern versions of word.</li>
                      </ol>
                    </div>

                    <div class="space-y-2">
                      <Button 
                        onclick={downloadHTMLFromInstructions}
                        class="w-full cursor-pointer"
                      >
                        <Download/> Download as HTML
                      </Button>
                      
                      <div class="flex gap-2">
                        <Button 
                          variant="outline"
                          onclick={goBackFromInstructions}
                          class="flex-1 cursor-pointer"
                        >
                          Back
                        </Button>
                        <Button 
                          variant="outline" 
                          onclick={() => {
                            downloadDialogOpen = false;
                            showDocxInstructions = false;
                          }}
                          class="flex-1 cursor-pointer"
                        >
                          Cancel
                        </Button>
                      </div>
                    </div>
                  </div>
                {/if}
              </Dialog.Content>
            </Dialog.Root>
          </div>
        </div>
      </Resizable.Pane>
    </Resizable.PaneGroup>
  {/if}
</div>