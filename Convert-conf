#!/usr/bin/env python3
"""
ka9q-radio Configuration Converter
Converts monolithic *.conf files to *.conf.d directory structure

Usage: python3 conf2confd.py input.conf [output_directory]
"""

import re
import os
import sys
import argparse
from pathlib import Path
from typing import Dict, List, Tuple

class Ka9qConfigConverter:
    def __init__(self):
        self.section_pattern = re.compile(r'^\[([^\]]+)\]', re.MULTILINE)
        self.comment_pattern = re.compile(r'^\s*[;#].*$', re.MULTILINE)
    
    def parse_config_file(self, file_path: str) -> Dict[str, str]:
        """Parse a ka9q-radio configuration file into sections"""
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        # Split into sections
        sections = {}
        current_section = None
        current_content = []
        
        lines = content.split('\n')
        for line in lines:
            # Check for section header
            section_match = re.match(r'^\s*\[([^\]]+)\]\s*$', line)
            if section_match:
                # Save previous section if exists
                if current_section is not None:
                    sections[current_section] = '\n'.join(current_content).rstrip()
                
                # Start new section
                current_section = section_match.group(1)
                current_content = [line]
            else:
                if current_section is not None:
                    current_content.append(line)
                else:
                    # Content before first section (comments, etc.)
                    if 'header' not in sections:
                        sections['header'] = []
                    sections['header'].append(line)
        
        # Save last section
        if current_section is not None:
            sections[current_section] = '\n'.join(current_content).rstrip()
        
        # Join header lines if present
        if 'header' in sections and isinstance(sections['header'], list):
            sections['header'] = '\n'.join(sections['header']).rstrip()
        
        return sections
    
    def generate_section_order(self, sections: Dict[str, str]) -> List[str]:
        """Generate logical order for sections"""
        order = []
        
        # Standard section order based on ka9q-radio documentation
        priority_sections = ['header', 'global', 'hardware', 'airspy', 'rx888', 'sdrplay', 'rtlsdr']
        
        # Add priority sections first
        for section in priority_sections:
            if section in sections:
                order.append(section)
        
        # Add remaining sections (channels, etc.)
        for section in sections:
            if section not in order:
                order.append(section)
        
        return order
    
    def create_confd_structure(self, sections: Dict[str, str], output_dir: str, base_name: str):
        """Create .conf.d directory structure"""
        confd_path = Path(output_dir) / f"{base_name}.conf.d"
        confd_path.mkdir(parents=True, exist_ok=True)
        
        section_order = self.generate_section_order(sections)
        
        for i, section_name in enumerate(section_order, 1):
            # Generate filename with numeric prefix
            filename = f"{i:02d}-{section_name}.conf"
            filepath = confd_path / filename
            
            content = sections[section_name]
            
            # Ensure section ends with newline
            if content and not content.endswith('\n'):
                content += '\n'
            
            with open(filepath, 'w', encoding='utf-8') as f:
                f.write(content)
            
            print(f"Created: {filepath}")
    
    def convert_file(self, input_file: str, output_dir: str = "."):
        """Convert a single .conf file to .conf.d structure"""
        input_path = Path(input_file)
        if not input_path.exists():
            raise FileNotFoundError(f"Input file not found: {input_file}")
        
        base_name = input_path.stem
        print(f"Converting: {input_file} -> {base_name}.conf.d")
        
        # Parse the configuration file
        sections = self.parse_config_file(input_file)
        
        if not sections:
            print("Warning: No sections found in configuration file")
            return
        
        print(f"Found {len(sections)} sections: {list(sections.keys())}")
        
        # Create the .conf.d directory structure
        self.create_confd_structure(sections, output_dir, base_name)
        
        # Create backup of original file
        backup_path = input_path.with_suffix(f"{input_path.suffix}.backup")
        if not backup_path.exists():
            import shutil
            shutil.copy2(input_path, backup_path)
            print(f"Backup created: {backup_path}")
        
        print(f"Conversion completed: {base_name}.conf.d")

def main():
    parser = argparse.ArgumentParser(
        description="Convert ka9q-radio *.conf files to *.conf.d directory structure",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python3 conf2confd.py radiod@airspy.conf
  python3 conf2confd.py radiod@airspy.conf /etc/radio
  python3 conf2confd.py *.conf
        """
    )
    
    parser.add_argument('input', nargs='+', help='Input .conf file(s) to convert')
    parser.add_argument('-o', '--output', default='.', 
                       help='Output directory (default: current directory)')
    parser.add_argument('-v', '--verbose', action='store_true',
                       help='Verbose output')
    
    args = parser.parse_args()
    
    converter = Ka9qConfigConverter()
    
    try:
        for input_file in args.input:
            if '*' in input_file:
                # Handle glob patterns
                from glob import glob
                for file_path in glob(input_file):
                    if file_path.endswith('.conf'):
                        converter.convert_file(file_path, args.output)
            else:
                converter.convert_file(input_file, args.output)
    
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()