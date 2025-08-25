# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a wireframe/prototype project for a Japanese real estate portal management system (不動産ポータル管理画面). The project consists of static HTML files with embedded CSS styles, creating mockup interfaces for various administrative functions.

## Project Structure

The project contains standalone HTML files, each representing a different page/feature of the management system:

### Core Management Pages
- `index.html` - Main dashboard/navigation page
- `company_register.html` - Company registration management
- `member_list.html` / `member_register.html` - Member management
- `inquiry_management.html` - Inquiry handling system

### Catalog Management
- `catalog_custom_home_list.html` / `catalog_custom_home_register.html` - Custom home catalog
- `catalog_new_house_list.html` / `catalog_new_house_register.html` - New house catalog
- `catalog_land_list.html` / `catalog_land_register.html` - Land catalog
- `catalog_request.html` - Catalog request management

### Tour & Event Management
- `tour_schedule_list.html` / `tour_schedule_register.html` - Tour scheduling
- `tour_reservation_list.html` / `tour_reservation_register.html` - Tour reservations
- `tour_user_reservation_list.html` - User reservation overview
- `all_events.html` - Event overview
- `event_setting.html` - Event configuration
- `event_requests.html` - Event request management

### Referral System (送客管理システム)
- `referral_list.html` / `referral_register.html` - Customer referral management
- `commission_settings.html` / `commission_list.html` - Commission management
- `estimate_list.html` / `estimate_history.html` - Estimate management and revision history

## Business Logic - Referral Management System

The referral management system implements a specific business model for real estate portal operations:

### Core Business Flow
1. Customer inquiries (tour reservations, event bookings, catalog requests) are collected
2. Customers are manually assigned to partner companies (house builders, real estate agencies)
3. Partner companies provide estimates to customers
4. Upon contract signing, commissions are calculated based on pre-set rates
5. Commission tracking and billing management

### Key Features
- **Customer Selection**: Choose from existing tour reservations, event bookings, or catalog requests
- **Multi-company Referrals**: Send customers to multiple partner companies simultaneously
- **Commission Rate Management**: Company-specific commission rates with historical tracking
- **Estimate Tracking**: Monitor estimates from partner companies with revision history
- **Revenue Tracking**: Automated commission calculation and billing status management

### Data Relationships
- Referrals link to existing customer data from tours/events/catalogs
- Commission rates are company-specific with effective date ranges
- Estimates can be revised multiple times with full audit trail
- Final contracts trigger automatic commission calculations

## Development Notes

### HTML/CSS Structure
- All pages use inline CSS within `<style>` tags
- Common design patterns include:
  - Fixed header with navigation (60px height)
  - Sidebar navigation (280px width)
  - Main content area with responsive layouts
  - Consistent color scheme: #2c2c2c (dark), #f9f9f9 (light background)

### Styling Conventions
- Font family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif
- Box-sizing: border-box applied globally
- Shadow effects for depth: box-shadow: 0 2px 10px rgba(0,0,0,0.1)

### No Build Process
This is a static HTML project with no build tools, package managers, or frameworks. All functionality is achieved through:
- Pure HTML structure
- Embedded CSS styles
- No external dependencies

## Common Tasks

### Adding a New Page
1. Create a new HTML file following the naming convention: `feature_action.html`
2. Copy the basic structure from an existing page
3. Maintain consistent header height (60px) and sidebar width (280px)
4. Use the established color scheme and typography

### Modifying Styles
- All styles are embedded within each HTML file's `<style>` tag
- Maintain consistency across pages when updating common elements
- Test responsive behavior by checking viewport meta tag settings