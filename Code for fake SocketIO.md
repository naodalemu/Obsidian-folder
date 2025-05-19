```
import { useState, useEffect } from "react";
import { useNavigate } from "react-router-dom";
import classes from "./OrderStatus.module.css";
import Backdrop from "./Backdrop";
import { FaWindowClose } from "react-icons/fa";
import MessageModal from "./MessageModal";

function OrderStatus() {
  const navigate = useNavigate();
  const [activeTab, setActiveTab] = useState("active");
  const [orders, setOrders] = useState([]);
  const [showDeleteWarning, setShowDeleteWarning] = useState(false);
  const [orderToDelete, setOrderToDelete] = useState(null);
  const [tooltipOrderId, setTooltipOrderId] = useState(null);
  const [tooltipText, setTooltipText] = useState("");
  const [tooltipPosition, setTooltipPosition] = useState({ x: 0, y: 0 });
  const [tableNumber, setTableNumber] = useState();
  const [arrivalStatus, setArrivalStatus] = useState(null);
  const [arrivalMessage, setArrivalMessage] = useState("");

  const filteredOrders = orders.filter((order) => {
    if (activeTab === "active")
      return ["ready", "pending", "processing"].includes(
        order.status.toLowerCase()
      );
    return ["completed", "canceled"].includes(order.status.toLowerCase());
  });

  const fetchOrders = async () => {
    try {
      const response = await fetch(
        `http://127.0.0.1:8000/api/orders/user?customer_ip=${localStorage.getItem(
          "customer_ip"
        )}&customer_temp_id=${localStorage.getItem("customer_generated_id")}`,
        {
          method: "GET",
          headers: {
            Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
          },
        }
      );

      if (!response.ok) {
        throw new Error("Failed to fetch orders");
      }

      const data = await response.json();

      const mappedOrders = data.orders.map((order) => ({
        id: order.id,
        date: new Date(order.order_date_time).toLocaleString(),
        status: order.order_status,
        payment_status: order.payment_status,
        arrived: !!order.arrived,
        total: parseFloat(order.total_price),
        is_remote: order.order_type === "remote",
        items: order.order_items.map((item) => ({
          id: item.id,
          name: item.menu_item.name,
          price: parseFloat(item.menu_item.price),
          quantity: item.quantity,
          image: `http://127.0.0.1:8000/storage/${item.menu_item.image}`,
        })),
      }));

      setOrders(mappedOrders);
    } catch (error) {
      console.error("Error fetching orders:", error);
    }
  };

  const fetchOrderStatuses = async () => {
    try {
      const response = await fetch("http://127.0.0.1:8000/api/order-status", {
        method: "GET",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("auth_token")}`,
        },
      });

      if (!response.ok) {
        throw new Error("Failed to fetch order statuses");
      }

      const statuses = await response.json();

      // Update the status of existing orders
      setOrders((prevOrders) =>
        prevOrders.map((order) => {
          const updatedStatus = statuses.find((s) => s.id === order.id);
          return updatedStatus
            ? { ...order, status: updatedStatus.status }
            : order;
        })
      );
    } catch (error) {
      console.error("Error fetching order statuses:", error);
    }
  };

  useEffect(() => {
    // Fetch orders initially
    fetchOrders();

    // Set up an interval to fetch order statuses every 5 seconds
    const intervalId = setInterval(() => {
      fetchOrderStatuses();
    }, 5000);

    // Clear the interval when the component unmounts
    return () => clearInterval(intervalId);
  }, []);

  return (
    <div className={classes.orderStatus}>
      <div className={classes.tabs}>
        <button
          className={`${classes.tabButton} ${
            activeTab === "active" ? classes.activeTab : ""
          }`}
          onClick={() => setActiveTab("active")}
        >
          Active Orders
        </button>
        <button
          className={`${classes.tabButton} ${
            activeTab === "history" ? classes.activeTab : ""
          }`}
          onClick={() => setActiveTab("history")}
        >
          Order History
        </button>
      </div>

      <div className={classes.ordersContainer}>
        {filteredOrders.map((order) => (
          <div key={order.id} className={classes.orderCard}>
            <div className={classes.orderHeader}>
              <div className={classes.orderInfo}>
                <h3>
                  Order #{order.id}{" "}
                  {order.payment_status.toLowerCase() === "completed" &&
                    "(Paid)"}
                </h3>
                <p>{order.date}</p>
              </div>
              <div className={classes.orderStatusOnCard}>
                <span
                  className={`${classes.statusBadge} ${
                    classes[order.status.toLowerCase()]
                  }`}
                >
                  {order.status}
                </span>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

export default OrderStatus;
```
