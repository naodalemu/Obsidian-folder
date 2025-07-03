```
const getAttendanceStatus = (staffId, staffShiftId, date) => {

const today = new Date().toISOString().split("T")[0]; // Get today's date in YYYY-MM-DD format

const formattedDate = formatDate(date);

  

if (formattedDate > today) {

// Do not show any attendance marks for future dates

return null;

}

  

const attendanceForStaff = attendance[staffId] || [];

const attendanceForShift = attendanceForStaff.find(

(record) =>

record.staff_shift_id === staffShiftId &&

record.scanned_at &&

record.scanned_at.startsWith(formattedDate)

);

  

if (formattedDate === today) {

// For today, only show present attendance

return attendanceForShift && attendanceForShift.status === "present"

? "present"

: null;

}

  

// For past dates, show both present and absent attendance

return attendanceForShift ? attendanceForShift.status : "absent";

};

  

const renderDayCell = (date) => {

const isToday = formatDate(date) === formatDate(new Date());

const isCurrentMonth = date.getMonth() === currentDate.getMonth();

const shiftsForDay = getShiftsForDate(date);

  

return (

<div

key={date.toString()}

className={`border border-gray-200 ${

viewMode === "week" ? "min-h-[200px]" : "min-h-[100px]"

} ${isCurrentMonth ? "bg-white" : "bg-gray-50"} ${

isToday ? "border-blue-500 border-2" : ""

}`}

>

<div className={`p-1 text-right ${isToday ? "bg-blue-50" : ""}`}>

<span className={`text-sm ${isToday ? "font-bold" : ""}`}>

{date.getDate()}

</span>

</div>

<div

className={`p-1 overflow-y-auto ${

viewMode === "week" ? "max-h-96" : "max-h-24"

}`}

>

{shiftsForDay.map((staffShift) => {

const shift = getShiftById(staffShift.shift_id);

const staffMember = getStaffById(staffShift.staff_id);

const colorClass = getShiftColor(staffShift.shift_id);

const attendanceStatus = getAttendanceStatus(

staffShift.staff_id,

staffShift.id, // Pass the staff_shift_id

date

);

const attendanceRecords = attendance[staffShift.staff_id]?.filter(

(record) =>

record.staff_shift_id === staffShift.id &&

record.scanned_at &&

record.scanned_at.startsWith(formatDate(date))

);

const attendanceForShift =

attendance[staffShift.staff_id]?.find(

(record) =>

record.staff_shift_id === staffShift.id &&

record.scanned_at &&

record.scanned_at.startsWith(formatDate(date))

) || 0;

  

// Aggregate data from clock_in and clock_out records

const lateMinutes = attendanceRecords

?.filter((record) => record.mode === "clock_in")

.reduce((acc, record) => acc + (record.late_minutes || 0), 0);

  

const earlyMinutes = attendanceRecords

?.filter((record) => record.mode === "clock_out")

.reduce((acc, record) => acc + (record.early_minutes || 0), 0);

  

const lateApproved = attendanceRecords?.some(

(record) => record.late_approved === 1

);

  

const earlyApproved = attendanceRecords?.some(

(record) => record.early_approved === 1

);

  

return (

<div

key={`${staffShift.id}`}

className={`text-xs p-1 mb-1 rounded border ${colorClass} truncate flex justify-between items-start`}

title={`${staffMember.name || "Staff"}: ${

shift.name || "Shift"

} (${staffShift.start_time || shift.start_time} - ${

staffShift.end_time || shift.end_time

})`}

>

<div>

<div className="font-semibold truncate">

{staffMember.name || "Staff"}

</div>

<div className="truncate">{shift.name || "Shift"}</div>

<div className="truncate">

{staffShift.start_time || shift.start_time} -{" "}

{staffShift.end_time || shift.end_time}

</div>

  

<div className="flex space-x-1">

{/* Approve Attendance */}

{/* Approve Attendance */}

{attendanceForShift &&

attendanceForShift.status === "present" &&

!isToday

? null

: formatDate(date) < formatDate(new Date()) && (

<button

onClick={() =>

approveAttendance(attendanceForShift.id)

}

className={`text-xs px-2 py-1 rounded ${

attendanceForShift?.approved_by_admin

? "border border-green-500 text-green-500"

: "border border-[#333] text-[#333]"

}`}

title="Approve Absent"

>

AA

</button>

)}

  

{/* Approve Late */}

{lateMinutes > 0 && (

<button

onClick={() =>

attendanceRecords && approveLate(attendanceRecords)

}

className={`text-xs px-2 py-1 rounded ${

lateApproved

? "border border-green-500 text-green-500"

: "border border-[#333] text-[#333]"

}`}

title="Approve Late"

>

AL

</button>

)}

  

{/* Approve Early */}

{earlyMinutes > 0 && (

<button

onClick={() =>

attendanceRecords && approveEarly(attendanceRecords)

}

className={`text-xs px-2 py-1 rounded ${

earlyApproved

? "border border-green-500 text-green-500"

: "border border-[#333] text-[#333]"

}`}

title="Approve Early"

>

AE

</button>

)}

</div>

</div>

  

<div className="flex flex-col items-center">

{attendanceStatus && (

<span

className={`text-md mb-2 ${

attendanceStatus === "present"

? "text-green-500"

: "text-red-500"

}`}

title={`Attendance: ${attendanceStatus}`}

>

{attendanceStatus === "present" ? (

<FaCheck />

) : (

<FaXmark />

)}

</span>

)}

<button

onClick={() => deleteStaffShift(staffShift.id)}

className="text-[#333] hover:text-[#444]"

title="Delete Shift"

>

<FiTrash2 />

</button>

</div>

</div>

);

})}

</div>

</div>

);

};
```


## Older Version

```
import { useState, useEffect } from "react";
import {
  FiChevronLeft,
  FiChevronRight,
  FiCalendar,
  FiTrash2,
} from "react-icons/fi";

const ShiftCalendarView = ({ reloadKey }) => {
  const [currentDate, setCurrentDate] = useState(new Date());
  const [staffShifts, setStaffShifts] = useState([]);
  const [staff, setStaff] = useState([]);
  const [shifts, setShifts] = useState([]);
  const [attendance, setAttendance] = useState({});
  const [loading, setLoading] = useState(true);
  const [viewMode, setViewMode] = useState("month"); // 'week' or 'month'

  const fetchData = async () => {
    try {
      const [staffResponse, shiftsResponse, assignmentsResponse] =
        await Promise.all([
          fetch(`${import.meta.env.VITE_BASE_URL}/api/admin/getStaff`, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
            },
          }),
          fetch(`${import.meta.env.VITE_BASE_URL}/api/shifts`, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
            },
          }),
          fetch(`${import.meta.env.VITE_BASE_URL}/api/staff-shifts`, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
            },
          }),
        ]);

      if (staffResponse.ok && shiftsResponse.ok && assignmentsResponse.ok) {
        const staffData = await staffResponse.json();
        const shiftsData = await shiftsResponse.json();
        const assignmentsData = await assignmentsResponse.json();

        setStaff(staffData);
        setShifts(shiftsData);
        setStaffShifts(assignmentsData);

        // Fetch attendance for all staff
        const attendanceData = {};
        for (const staffMember of staffData) {
          const attendanceResponse = await fetch(
            `${import.meta.env.VITE_BASE_URL}/api/attendance/${staffMember.id}`,
            {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
              },
            }
          );
          if (attendanceResponse.ok) {
            const attendanceResult = await attendanceResponse.json();
            attendanceData[staffMember.id] = attendanceResult.attendance;
          }
        }
        setAttendance(attendanceData);
      }
    } catch (error) {
      console.error("Error fetching data:", error);
    } finally {
      setLoading(false);
    }
  };

  const deleteStaffShift = async (id) => {
    try {
      const response = await fetch(
        `${import.meta.env.VITE_BASE_URL}/api/staff-shifts/${id}`,
        {
          method: "DELETE",
          headers: {
            Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
          },
        }
      );

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || "Failed to delete staff shift.");
      }

      // Remove the deleted shift from the state
      setStaffShifts((prevShifts) =>
        prevShifts.filter((shift) => shift.id !== id)
      );
    } catch (error) {
      console.error("Error deleting staff shift:", error);
      alert(
        error.message || "An error occurred while deleting the staff shift."
      );
    }
  };

  useEffect(() => {
    fetchData();
  }, [reloadKey]);

  const getAttendanceStatus = (staffId, date) => {
    const attendanceForStaff = attendance[staffId] || [];
    const attendanceForDate = attendanceForStaff.find(
      (record) => record.scanned_at && record.scanned_at.startsWith(date)
    );
    return attendanceForDate ? attendanceForDate.status : "absent";
  };

  const renderDayCell = (date) => {
    const isToday = formatDate(date) === formatDate(new Date());
    const isCurrentMonth = date.getMonth() === currentDate.getMonth();
    const shiftsForDay = getShiftsForDate(date);

    return (
      <div
        key={date.toString()}
        className={`border border-gray-200 ${
          viewMode === "week" ? "min-h-[200px]" : "min-h-[100px]"
        } ${isCurrentMonth ? "bg-white" : "bg-gray-50"} ${
          isToday ? "border-blue-500 border-2" : ""
        }`}
      >
        <div className={`p-1 text-right ${isToday ? "bg-blue-50" : ""}`}>
          <span className={`text-sm ${isToday ? "font-bold" : ""}`}>
            {date.getDate()}
          </span>
        </div>
        <div
          className={`p-1 overflow-y-auto ${
            viewMode === "week" ? "max-h-96" : "max-h-24"
          }`}
        >
          {shiftsForDay.map((staffShift) => {
            const shift = getShiftById(staffShift.shift_id);
            const staffMember = getStaffById(staffShift.staff_id);
            const colorClass = getShiftColor(staffShift.shift_id);
            const attendanceStatus = getAttendanceStatus(
              staffShift.staff_id,
              formatDate(date)
            );

            return (
              <div
                key={`${staffShift.id}`}
                className={`text-xs p-1 mb-1 rounded border ${colorClass} truncate flex justify-between items-start`}
                title={`${staffMember.name || "Staff"}: ${
                  shift.name || "Shift"
                } (${staffShift.start_time || shift.start_time} - ${
                  staffShift.end_time || shift.end_time
                })`}
              >
                <div>
                  <div className="font-semibold truncate">
                    {staffMember.name || "Staff"}
                  </div>
                  <div className="truncate">{shift.name || "Shift"}</div>
                  <div className="truncate">
                    {staffShift.start_time || shift.start_time} -{" "}
                    {staffShift.end_time || shift.end_time}
                  </div>
                </div>
                <div className="flex flex-col items-center">
                  <button
                    onClick={() => deleteStaffShift(staffShift.id)}
                    className="text-[#333] hover:text-[#444]"
                    title="Delete Shift"
                  >
                    <FiTrash2 />
                  </button>
                  <span
                    className={`text-lg ${
                      attendanceStatus === "present"
                        ? "text-green-500"
                        : "text-red-500"
                    }`}
                    title={`Attendance: ${attendanceStatus}`}
                  >
                    {attendanceStatus === "present" ? "✔" : "✖"}
                  </span>
                </div>
              </div>
            );
          })}
        </div>
      </div>
    );
  };

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="text-gray-500">Loading calendar...</div>
      </div>
    );
  }

  const days = viewMode === "month" ? getMonthDays() : getWeekDays();
  const dayNames = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];

  return (
    <section className="p-6">
      <div>
        <div className="flex justify-between items-center mb-6">
          <h1 className="text-lg font-bold text-gray-800"></h1>

          <div className="flex items-center space-x-2">
            <button
              onClick={navigateToday}
              className="bg-white border border-gray-300 rounded-md px-3 py-1 text-sm"
            >
              Today
            </button>

            <button
              onClick={toggleViewMode}
              className="bg-white border border-gray-300 rounded-md px-3 py-1 text-sm flex items-center"
            >
              <FiCalendar className="mr-1" />
              {viewMode === "month" ? "Week View" : "Month View"}
            </button>

            <div className="flex items-center">
              <button
                onClick={navigatePrevious}
                className="bg-white border border-gray-300 rounded-l-md px-2 py-1"
              >
                <FiChevronLeft />
              </button>
              <div className="bg-white border-t border-b border-gray-300 px-3 font-medium">
                {viewMode === "month"
                  ? currentDate.toLocaleDateString("en-US", {
                      month: "long",
                      year: "numeric",
                    })
                  : `${days[0].toLocaleDateString("en-US", {
                      month: "short",
                      day: "numeric",
                    })} - ${days[6].toLocaleDateString("en-US", {
                      month: "short",
                      day: "numeric",
                    })}`}
              </div>
              <button
                onClick={navigateNext}
                className="bg-white border border-gray-300 rounded-r-md px-2 py-1"
              >
                <FiChevronRight />
              </button>
            </div>
          </div>
        </div>

        <div className="bg-white rounded-lg shadow-md overflow-hidden">
          <div className="grid grid-cols-7">
            {dayNames.map((day) => (
              <div
                key={day}
                className="p-2 text-center font-medium bg-gray-50 border-b border-gray-200"
              >
                {day}
              </div>
            ))}
          </div>

          <div
            className={`grid grid-cols-7 ${
              viewMode === "month" ? "grid-rows-6" : "grid-rows-1"
            }`}
          >
            {days.map((day) => renderDayCell(day))}
          </div>
        </div>
      </div>
    </section>
  );
};

export default ShiftCalendarView;
```

