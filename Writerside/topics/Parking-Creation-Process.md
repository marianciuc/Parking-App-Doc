# Parking Management System API Documentation

This documentation provides comprehensive guidance for developers to interact with the **Parking Management System** via
REST APIs. It includes endpoint descriptions, request/response formats, role-based permission details, and use case
scenarios. The goal is to support frontend developers in seamlessly integrating parking management functionalities into
their applications.

---

## 1. Introduction

The **Parking Management System API** offers secure and role-based access to manage parking lots efficiently. The system
is tailored for two primary roles: **Admins** and **Parking Owners**, ensuring that actions are authorized and access to
resources remains secure. The API leverages robust authentication mechanisms to protect sensitive data.

### Key Features:

- **Parking Lot Management:** Create, update, delete, and approve parking lots.
- **Advanced Search & Filtering:** Query parking data with customizable filters.
- **Tags Management:** Assign, update, and categorize parking lots using tags.
- **Access Control:** Manage inclusivity using whitelists/blacklists.
- **Operational Updates:** Modify parking lot data such as opening hours and availability.

---

## 2. Creating a Parking Lot

### Purpose {id="purpose_1"}

The `POST /api/v1/parking/creation` endpoint allows authorized users to register new parking lots in the system. The new
parking lot will remain in a `CREATION_PROCESS` state until explicitly approved.

### Required Parameters

To create a parking lot, the following fields must be included in the body of the request:

| **Field**                  | **Type**  | **Description**                           | **Required** |
|----------------------------|-----------|-------------------------------------------|--------------|
| `name`                     | `String`  | Name of the parking lot.                  | Yes          |
| `address.city`             | `String`  | City where the parking lot is located.    | Yes          |
| `address.countryCode`      | `String`  | Country code in ISO format (e.g., "US").  | Yes          |
| `address.latitude`         | `Float`   | Latitude of the location.                 | Yes          |
| `address.longitude`        | `Float`   | Longitude of the location.                | Yes          |
| `isBelongToAnyInstitution` | `Boolean` | Indicates if owned by an institution.     | No           |
| `institutionName`          | `String`  | Specifies institution name if applicable. | Conditional  |

### Workflow {id="workflow_1"}

1. **Input Preparation:** Ensure all required parameters are provided and formatted correctly.
2. **Send Request:** Submit a `POST` request to `/api/v1/parking/creation` along with the payload.
3. **Response:** Upon success, a `201 Created` response is returned with parking lot details.

### Common Responses:

- **Success:** `201 Created`  
  A new parking lot entity is returned.
- **Permission Denied:** `403 Forbidden`  
  The user does not have a `PARKING_OWNER` role.

---

## 3. Updating Parking Lot Status

### Purpose {id="purpose_2_1"}

Parking lot statuses can transition between `CREATION_PROCESS`, `ACTIVE`, or `REJECTED` to control visibility and
access.

### Workflow

1. **Endpoint:** Send a `PUT` request to `/api/v1/parking/{parkingId}/status` with the desired status.
2. **Approval:** Only admins are allowed to change statuses.
3. **Response:** Successful requests return an updated status confirmation.

### Example Request:

```json
{
  "status": "ACTIVE"
}
```

---

## 4. Managing Tags

Tags enable categorization of parking lots for enhanced searchability. Admins can manage tags at both the parking lot
and system levels.

### Example Use-Cases

- Categorizing lots by features such as `Covered`, `Free Wi-Fi`, or `EV Charging`.
- Assigning tags for filtering and search purposes.

#### **Adding Tags to a Parking Lot**

Send a `PUT` request to `/api/v1/parking/{parkingId}/tags`:

```json
{
  "tags": [
    "tag_id_1",
    "tag_id_2"
  ]
}
```

#### **Creating New Tags**

Admins may create new tags using the `POST /api/v1/tags` endpoint. Required fields:

| **Field**     | **Type** | **Description**         | **Required** |
|---------------|----------|-------------------------|--------------|
| `name`        | `String` | Name of the tag.        | Yes          |
| `description` | `String` | Description of the tag. | Yes          |

#### **Deleting Tags**

Issue a `DELETE /api/v1/tags/{tagId}` to remove tags from lots.

---

## 5. Deleting Parking Lots

### Purpose {id="purpose_2_2"}

Delete unused or irrelevant parking lots using the `DELETE /api/v1/parking/{parkingId}` endpoint.

### Restrictions:

- **Role-Based Access:** Only admins or lot owners can perform deletions.
- **Active Usage:** Lots with active sessions cannot be deleted (`409 Conflict`).

---

## 6. Search and Filters

### Purpose {id="purpose_2"}

Enable users to search for parking lots using query parameters or advanced filters.

### Query Parameters:

| **Parameter**   | **Type**  | **Description**                                           |
|-----------------|-----------|-----------------------------------------------------------|
| `page`          | `Integer` | Pagination parameter: current page number.                |
| `size`          | `Integer` | Defines the number of results per page.                   |
| `includeHidden` | `Boolean` | Includes parking lots with the status `CREATION_PROCESS`. |

### Advanced Filters:

| **Field**       | **Type** | **Description**                               |
|-----------------|----------|-----------------------------------------------|
| `name`          | `String` | Parking lot name (e.g., "Central Park").      |
| `city`          | `String` | Lots located in this city (e.g., "New York"). |
| `recordStatus`  | `Enum`   | Active, Deleted, or Hidden (see Enums below). |
| `parkingStatus` | `Enum`   | Defines operational parking lot status.       |

---

## Enumeration Definitions

Enums are used throughout the API to define controlled values.

### 1. **AccessType Enum**

Defines who can access a parking lot:

| **Value**        | **Description**                                |
|------------------|------------------------------------------------|
| `OPEN`           | Open to any authorized user.                   |
| `WHITELIST_ONLY` | Access restricted to listed users or vehicles. |

### 2. **ParkingStatus Enum**

Categorizes the current state of the parking lot:

| **Value**          | **Description**                    |
|--------------------|------------------------------------|
| `CREATION_PROCESS` | Pending approval after creation.   |
| `TEMPORARY_CLOSED` | Temporarily unavailable.           |
| `OPEN`             | Operational and accepting parkers. |
| `CLOSED`           | Closed during non-operating hours. |

### 3. **ParkingType Enum**

Describes the type of parking lot:

| **Value**     | **Description**                                    |
|---------------|----------------------------------------------------|
| `OUTDOOR`     | Open-air parking.                                  |
| `INDOOR`      | Covered or garage parking.                         |
| `MULTI_LEVEL` | Multi-tiered parking facilities.                   |
| `STREET`      | Curbside or public street parking.                 |
| `RESIDENTIAL` | Reserved for residents of a community or building. |

### 4. **RecordStatus Enum**

Status of parking data records:

| **Value** | **Description**                             |
|-----------|---------------------------------------------|
| `ACTIVE`  | Lot is available for use.                   |
| `HIDDEN`  | Lot is hidden from standard search results. |
| `DELETED` | Lot is permanently removed from the system. |

---

This guide ensures that developers have a deep understanding of all the critical functionalities supported by the *
*Parking Management System API**. For additional queries or feature enhancements, reach out to the system administrator
or API support team.